# Speculative Execution Side Channel Attacks (Spectre)

Side-Channel Attacks are a class of exploits that use speculative execution as a mean to exfiltrate users' data by tricking the processor into executing an illegal sequence of instruction (also called *Transient Instructions*) by exploiting intrinsic flaws of the functional units of the processor.

This report will cover the "original" Side-Channel Attacks, namely Spectre-v1, Spectre-v2, and Meltdown.

## Overview

**Speculative Execution Side Channel Attacks** (also known as **Spectre**) are a family of exploits that rely on the fact that some conditional instructions in modern CPUs use specific hardware components called *Indirect Branch Predictors*, which are fundamentally flawed. These functional units essentially try to guess the position of the Program Counter after a branching point and start executing instructions *speculatively*, that is without knowing if they'll actually be valid.

The IBP is mainly useful when the branch depends on a value that's not present in the processor's cache, and needs to be fetched from memory: since this operation is costly, we can fill the clock cycles that would otherwise be wasted while waiting for this value by executing ahead on one branch. The IBP roughly operates as follow: 

1. When a branching point that depends on a value not present in the register file or in the cache is reached, the IBP checkpoints the state of the CPU registers;
2. The most likely branch to be taken is chosen <!--based on ??--> and speculative execution starts;
3. When the value the conditional instruction depends on arrives from memory, the truth value of the operation is compared to the speculated one:
   - If the predictor was right, the execution continues as normal;
   - If the predictor was wrong, the processor's state is reverted to the checkpoint and execution continues from there.

Since in the latter case the degradation of performance compared to idling is negligible, this operation has obvious benefits, however we must note that it violates the causality principle and undermines many assumptions that are commonly taken for granted. Take this fragment of code as an example:

```c
if (x < array1_size)
    y = array2[array1[x] * 4096];
```

Rationally, there should be no way for an indirect reference outside of `array1` to be used, as we know for a fact that the branch will never be taken. But what if the Branch Predictor decides to speculatively execute the instructions in the branch anyway? In this case, it was commonly believed that restoring the checkpoint would be enough to avoid any potential leakage, since the state of the processor was essentially reset to an earlier point in time.

The critical aspect of this scenario is *how* the branch is chosen: if we could train the Predictor to assume that the branch is taken, we could then trick it into accessing a location in the heap outside the given memory map by manipulating the value of `x`.

Finally, the checkpoint mechanism does not account for potential manipulations of the cache by Transient Instructions, and since the cache is vulnerable to timing attacks, whatever's being cached can be recovered by carefully crafted malicious code. 

### Bounds Check Bypass Attack

The **Bounds Check Bypass** (Spectre-v1) attack exploits this vulnerability in the IBP to allow the attacker to leak information by reading the contents of the cache after training it in speculatively executing a sequence of illegal instructions involving memory accesses.

The execution of the attack is split into several phases:

1. **Set-up** - The attacker sets up a specific attack scenario, namely 
2. **Training** - The attacker trains the branch predictor into mispredicting the target of the branch
3. **Eviction** - The attacker evicts the value the branch depends on from the cache in order to force a miss and the start of the speculative execution of transient instructions
4. **Retrieval** - The attacker retrieves the contents of the cache with a timing attack and tries to recover the leaked data from the context

We will discuss each phase in more detail when commenting the code.

#### Proof-of-Concept

The original Spectre paper gives a

### Spectre-v2


## Use case: the NetSpectre attack

https://martinschwarzl.at/media/files/netspectre.pdf

## Notable Examples of Vulnerable CPUs

The Spectre vulnerabilities seem to be an industry-wide issue, as it affects most of the prominent high-performance CPUs on the market. The Linux Kernel documentation cites the following:

- Intel Core, Atom, Pentium, and Xeon processors
- AMD Phenom, EPYC, and Zen processors
- IBM POWER and zSeries processors
- Higher end ARM processors
- Apple CPUs
- Higher end MIPS CPUs

It shall be noted that, since the discovery of the vulnerabilities, most silicon vendors seem to have implemented hardware or microcode fixes for Spectre.

### Spectre and Simultaneous Multithreading (SMT)

Simultaneous Multithreading (SMT), also known as Hyperthreading, is a technique used to enhance the multiprocessing capabilities of modern CPUs by assigning two logical processors to a single physical core. 

While SMT is beneficial for performance in highly parallel workloads, it has several drawbacks related to latency and determinism due to the fact that, intrinsically, the logical processors share the same underlying hardware components.

This also poses some security concerns as, for example, the IBP is shared among the logical processors. This obviously extends the attack surface as it breaks the assumption that processes running on different cores can't be snooped upon using Spectre.

Since this is an intrinsic hardware feature, there are essentially no mitigations in place for this specific scenario on affected CPUs besides disabling SMT altogether. It shall be noted, however, that mitigations that render Spectre attacks ineffective will work even across logical cores.

More modern CPUs use what Intel calls the Single-Threaded IBP (STIBP) to resolve this issue on the hardware level. The STIBP is a non-shared component that physically isolates the BPBs for each logical processor. <!-- TODO: Verify -->

## Mitigations

Mitigations for Spectre attacks do exist in both software (Retpolines), firmware (Microcode Updates), and hardware (IBRS). In this section we will discuss a few specific examples, why they are relevant

### Impact of Mitigations on Performance

https://www.phoronix.com/review/3-years-specmelt/9

Seems to be quite relevant for older processors. Not so sure about newer ones with actual hw fixes.




### CVE List

| CVE           |                         |            |
| :------------ | ----------------------- | ---------- |
| CVE-2017-5753 | Bounds Check Bypass     | Spectre-v1 |
| CVE-2017-5715 | Branch Target Injection | Spectre-v2 |

## Sources

- Intel. (2021, May 26). Speculative Execution Side Channel Mitigations. www.intel.com. Retrieved November 8, 2023, from [https://www.intel.com/content/www/us/en/developer/articles/technical/software-security-guidance/technical-documentation/speculative-execution-side-channel-mitigations.html](https://www.intel.com/content/www/us/en/developer/articles/technical/software-security-guidance/technical-documentation/speculative-execution-side-channel-mitigations.html)
- IBM. (2021, September 15). How-to disable/enable Spectre/Meltdown mitigaton on POWER9 Systems. ibm.com. Retrieved November 8, 2023, from [https://www.ibm.com/support/pages/node/715841](https://www.ibm.com/support/pages/node/715841)
- FreeBSD. (2022, February 10). SpeculativeExecutionVulnerabilities - FreeBSD Wiki. wiki.freebsd.org. Retrieved November 8, 2023, from [https://wiki.freebsd.org/SpeculativeExecutionVulnerabilities](https://wiki.freebsd.org/SpeculativeExecutionVulnerabilities)
- Linux Kernel Developers. (n.d.). Spectre Side Channels — The Linux Kernel  documentation. www.kernel.org. Retrieved November 9, 2023, from [https://www.kernel.org/doc/html/latest/admin-guide/hw-vuln/spectre.html](https://www.kernel.org/doc/html/latest/admin-guide/hw-vuln/spectre.html)