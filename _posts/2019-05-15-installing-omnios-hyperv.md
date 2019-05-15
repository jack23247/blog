---
title: "Installing OmniOS on Hyper-V... or not?"
date: 2019-05-15T11:30:00+01:00
categories:
 - sysadm
tags:
 - virtualization
 - unix
 - microsoft
 - hyperv
---

> Update: OmniOS works perfectly on Hyper-V but somehow **needs two or more VCPUS to boot**, see (https://github.com/omniosorg/illumos-omnios/issues/459

OmniOS Community Edition is a Unix distribution based on Illumos, the modern and open OpenSolaris based kernel. OmniOS is blazingly fast and packs a lot of features but, most importantly, it supports a wide range of hypervisors, [including Hyper-V](<https://github.com/omniosorg/omnios-build/blob/r151030/doc/ReleaseNotes.md>).

To try OmniOS out, i created a Gen2 VM on Hyper-V and attempted to install the operating system. Upon booting it drops me to a nice boot menu. If i try booting the installer though, it hangs while loading the kernel with this message:

```
SunOS Release 5.11 Version omnios-r151030-f1189fc02c 64-bit
Copyright (c) 1983, 2010, Oracle and/or its affiliates. All rights reserved.
Copyright (c) 2017-2019 OmniOS Community Edition (OmniOSce) Association.

panic[cpu0]/thread=fffffffffbc490a0: microfind: could not calibrate delay loop
```

This looks like a timing error on first sight.

Let's look into the Illumos source tree, which is hosted on [GitHub](<https://github.com/illumos/illumos-gate/>). If we take a peek in [`microfind.c`](<https://github.com/illumos/illumos-gate/blob/master/usr/src/uts/i86pc/io/microfind.c>)'s (well made) comments, we can easily understand what is going on:

```c
/*
 * The microfind() routine is used to calibrate the delay provided by
 * tenmicrosec().  Early in boot gethrtime() is not yet configured and
 * available for accurate delays, but some drivers still need to be able to
 * pause execution for rough increments of ten microseconds.  To that end,
 * microfind() will measure the wall time elapsed during a simple delay loop
 * using the Intel 8254 Programmable Interval Timer (PIT), and attempt to find
 * a loop count that approximates a ten microsecond delay.
 *
 * This mechanism is accurate enough when running unvirtualised on real CPUs,
 * but is somewhat less efficacious in a virtual machine.  In a virtualised
 * guest the relationship between instruction completion and elapsed wall time
 * is, at best, variable; on such machines the calibration is merely a rough
 * guess.
 */
```

Ok, so `microfind.c` must be unable to use the Intel 8254 PIT to calibrate the kernel timing: of course it could not go on loading!

Let's see what Microsoft has to say about this:

> Because the PIT timer is not present in Generation 2 Virtual Machines, network connections to the PxE TFTP server can be prematurely terminated and prevent the bootloader from reading Grub configuration and loading a kernel from the server.
>
> *[Microsoft Docs](<https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/best-practices-for-running-linux-on-hyper-v>)*

More than a *rough guess*, it's an *unsupported guess*: looks like we won't be able to use Gen2 virtual machines, let's try and switch to Gen1.

Upon switching to a Gen1, or *legacy*, virtual machine, I tried to boot into the OS once more. This time the problem is more subtle and way less informative:

```
SunOS Release 5.11 Version omnios-r151030-f1189fc02c 64-bit
Copyright (c) 1983, 2010, Oracle and/or its affiliates. All rights reserved.
Copyright (c) 2017-2019 OmniOS Community Edition (OmniOSce) Association.
Configuring devices.
```

And it just stands there forever.

If we enable verbose boot, this is what we get:

```
hyperv_identify: Checking Hyper-V features... 
do_cpuid: cpuid leaf=0x40000000, eax=0x40000006, ebx=0x7263694d, ecx=0x666f736f, edx=0x76482074 
do_cpuid: cpuid leaf=0x40000001, eax=0x31237648, ebx=0x00000000, ecx=0x00000000, edx=0x00000000 
do_cpuid: cpuid leaf=0x40000003, eax=0x00002e7f, ebx=0x003880b0, ecx=0x00000002, edx=0x0Obed7b2
do_cpuid: cpuid leaf=0x40000002, eax=0x000047a7, ebx=0x000a0000, ecx=0x00000000, edx=0x00000001
Hyper-V Version: 10.0.18343 [SP0]
Features=0x2e7f<VPRUNTIME,TMREFCHT,SYNIC,SYHTM,RPIC,HYPERCRLL,VPINDEX,REFTSC,IDLE,TMFREQ> 
  Features1=0x3880b0<PostMessages,SignalEvents>
  Features2[PM]=0x0 [C2]  
  Features3=0xbed7b2<DEBUG,XMMHC,IDLE,HUMR,TMFREQ,SYNCMC,CRRSH,HPIEP>
do_cpuid: cpuid leaf=0x40000004, eax=0x00020c2c, ebx=0x0000Offf, ecx=0x0000002e, edx=0x00000000
hyperv_identify: Recommends: 00020c2c 0000Offf
do_cpuid: cpuid leaf=0x40000005, eax=0x000000f0, ebx=0x00000200, ecx=0x00000324, edx=0x00000000
hyperv_identify: Limits: Vcpu:240 Lcpu:512 Int:804
do_cpuid: cpuid leaf=0x40000006, eax=0x0000000f, ebx=0x00000000, ecx=0x00000000 , edx=0x00000000 
hyperv_identify: HW Features: 0000000f, RMD: 00000000
hv_vmbus0: hypercall_create: Enabling Hypercall interface...
hv_vmbus0: hypercall_create: Current Hypercall MSR: 0x0
hv_vmbus0: hypercall_create: Hypercall Page allocation done: 0xfffffe01e718d000 
hv_vmbus0: hypercall_create: Programming Hypercall MSR: 0x513372161
hv_vmbus0: hypercall_create: Verified Hypercall MSR: 0x513372161
hv_vmbus0: hypercall_create: Enabling Hypercall interface - SUCCESS !

```

And the OS does not load any further.

Looks like the version of Hyper-V I'm running (Windows 10 1903 18343.1) is not well supported by OmniOS at this point. I could revert to VirtualBox but this means dropping Hyper-V, as the hv interface, even for the root partition, is locked out.
