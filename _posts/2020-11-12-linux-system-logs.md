---
title: "Analyzing the various Linux system logs"
date: 2020-11-12T03:10:00+01:00
categories:
 - sysadm
tags:
 - linux
 - system
 - logs
 - admin
 - kernel
 - systemd
---

Yesterday I've been asked by a friend who is currently working on an autonomous driving platform, running ROS on Ubuntu Linux, how to understand what caused the system to hang: she told me that the computer controlling the platform suddenly had a stroke (that luckily had no adverse consequences, but could have been way worse) and that she seriously wanted to find out what happened.

Unfortunately, there is no simple answer: while definitely a great feat of engineering, the Linux kernel is not infallible. Being so close to the hardware means having a complex architecture exposed to all sorts of dangers: the kernel responsible for an incredible amount of critical tasks that could fail and create all sorts of problems. Moreover, there is also a myriad of problems that could be caused by misbehaving hardware devices. 

Presented here is a synthesis of the most useful log files, and of the many ways the provided tools can be exploited to really understand what kind of bad trip the system was having while trying to kill you.

## The log files

Logs are kept in `\var\log`, here's some notable examples:

| Name       | Description                                                          |
| ---------- | -------------------------------------------------------------------- |
| `boot.log` | A log that records everything that happened during the boot sequence |
| `dmesg`    | A log file used by the omonimous tool                                |
| `kern.log` | A specific log for kernel events                                     |
| `syslog`   | A global system log that records virtually everything                |

Let's try and analyze all of them.

### `boot.log`

The several copies of `boot.log` kept here represent an exact copy of the boot console messages, plus some timestamping. They're useful to determine the root cause of a boot problem, be it a delay or hanging before reaching the multiuser target (in case we're running `systemd`) or the login prompt.

<img src="https://raw.githubusercontent.com/jack23247/blog/master/img/case_of_systemd.jpg" alt="incaseofsystemd" style="zoom:75%;" />

Here's an excerpt from my `boot.log.1`:

```shell
$ sudo head -n 20 boot.log.1
[sudo] password for quartz: 
------------ Wed Nov 11 15:30:34 CET 2020 ------------
[  OK  ] Started Show Plymouth Boot Screen.
[  OK  ] Started Forward Password Requests to Plymouth Directory Watch.
[  OK  ] Reached target Local Encrypted Volumes.
[  OK  ] Found device TOSHIBA_THNSNH128GCST 1.
         Starting File System Check on /dev/disk/by-uuid/DFB7-381A...
[  OK  ] Listening on Load/Save RF Kill Switch Status /dev/rfkill Watch.
         Starting Load/Save RF Kill Switch Status...
[  OK  ] Started File System Check Daemon to report status.
[  OK  ] Finished File System Check on /dev/disk/by-uuid/DFB7-381A.
         Mounting /boot/efi...
[  OK  ] Mounted /boot/efi.
[  OK  ] Reached target Local File Systems.
         Starting Load AppArmor profiles...
         Starting Set console font and keymap...
         Starting Tell Plymouth To Write Out Runtime Data...
         Starting QEMU KVM preparation - module, ksm, hugepages...
         Starting Create Volatile Files and Directories...
```

It should be fairly readable, albeit a little hard to search through. It shouldn't be very long though, so you should be fine with a quick glance.

### `dmesg`

`dmesg` is the *kernel ring buffer* or, in layman's terms, a memory region in the kernel that keeps track of its activity: since the very beginning of the boot sequence, just out of the bootloader, `dmesg` records and timestamps everything the kernel and its affiliates do. 

#### The message format

First of all, let's take a look at the structure of the messages:

```shell
$ dmesg | head -n 10
[    0.000000] microcode: microcode updated early to revision 0x2f, date = 2019-02-17
[    0.000000] Linux version 5.4.74-rt41 (quartz@melchior) (gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)) #1 SMP PREEMPT_RT Tue Nov 3 21:55:23 CET 2020
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-5.4.74-rt41 root=UUID=567d8513-a279-4f63-b486-af9d2897a3be ro quiet splash vt.handoff=7
[    0.000000] KERNEL supported cpus:
[    0.000000]   Intel GenuineIntel
[    0.000000]   AMD AuthenticAMD
[    0.000000]   Hygon HygonGenuine
[    0.000000]   Centaur CentaurHauls
[    0.000000]   zhaoxin   Shanghai  
[    0.000000] Disabled fast string operations
$ dmesg | tail -n 10
[17885.157559] ata1.00: ACPI cmd ef/10:03:00:00:00:a0 (SET FEATURES) filtered out
[17885.158069] ata1.00: configured for UDMA/100
[17885.255281] usb 1-1.3: reset full-speed USB device number 3 using ehci-pci
[17885.701050] psmouse serio1: synaptics: queried max coordinates: x [..5472], y [..4448]
[17885.891382] OOM killer enabled.
[17885.891424] ACPI: \_SB_.PCI0.LPC_.EC__.BAT1: docking
[17885.900415] ACPI: \_SB_.PCI0.LPC_.EC__.BAT1: Unable to dock!
[17885.891392] Restarting tasks ... done.
[17886.607287] PM: suspend exit
[17886.858980] e1000e: enp0s25 NIC Link is Down

```

As you can see, the messages are timestamped, but by default `dmesg` outputs them in a tick-based way. By leveraging some of the flags, we can basically print them out however we want:

```shell
$ dmesg -T | tail -n 1
[mer nov 11 20:52:06 2020] IPv6: ADDRCONF(NETDEV_CHANGE): wlp3s0: link becomes ready
```

By using the `-T` flag, for example, we can ask for a human-readable time/date instead of the incremental timestamp. Be noted that **the date might not be accurate if the system has gone to sleep**, so keep it in mind when looking at `dmesg` output on portable computers or home workstations.

#### Filtering output by type

`dmesg` is a treasure trove of informations but the output might look a bit intimidating at first, and it's a pain to search through: you might be looking for a specific kind of message but be unable to `grep` through them, because there are no recurrent keywords. Once again, `dmesg` comes to rescue: the `-f, --facility` and `-l, --level`  flags with one of the following parameters (from `dmesg --help`) might help you look for specific things:

```shell
$ dmesg --help | tail -n 21
Supported log facilities:
    kern - kernel messages
    user - random user-level messages
    mail - mail system
  daemon - system daemons
    auth - security/authorization messages
  syslog - messages generated internally by syslogd
     lpr - line printer subsystem
    news - network news subsystem

Supported log levels (priorities):
   emerg - system is unusable
   alert - action must be taken immediately
    crit - critical conditions
     err - error conditions
    warn - warning conditions
  notice - normal but significant condition
    info - informational
   debug - debug-level messages

For more details see dmesg(1).
```

  For example, we might display the latest available error message this way:

```shell
$ dmesg -l err | tail -n 1
[17885.900415] ACPI: \_SB_.PCI0.LPC_.EC__.BAT1: Unable to dock!
```

...and this tells you that I've undocked my ThinkPad 😀.

#### Other combinations

`dmesg` is an extremely powerful and flexible tool: please take a cursory look at its `--help` flag and manual page, as I'm sure you'll be able to find someway to do whatever you need.

### `kern.log`

While `dmesg` handles kernel logs dynamically, it's mostly useful to analyze the current session. For past kernel message it might be worth it to look at the `kern.log`. Besides, the output format of `dmesg` is much more civilized. Here's an excerpt from `kern.log`:

```shell
$ cat kern.log | tail -n 5
Nov 11 20:55:43 melchior kernel: [19306.237788] 002: wlp3s0: associated
Nov 11 20:55:43 melchior kernel: [19306.389998] 003: IPv6: ADDRCONF(NETDEV_CHANGE): wlp3s0: link becomes ready
Nov 11 23:36:00 melchior kernel: [28922.398944] 000: audit: type=1326 audit(1605134160.384:35): auid=1000 uid=1000 gid=1000 ses=3 subj==snap.snap-store.ubuntu-software (enforce) pid=1864 comm="snap-store" exe="/snap/snap-store/481/usr/bin/snap-store" sig=0 arch=c000003e syscall=93 compat=0 ip=0x7f06966c6527 code=0x50000
Nov 12 00:00:44 melchior kernel: [30406.721556] 000: audit: type=1400 audit(1605135644.648:36): apparmor="DENIED" operation="capable" profile="/usr/sbin/cups-browsed" pid=14896 comm="cups-browsed" capability=23  capname="sys_nice"
Nov 12 00:04:33 melchior kernel: [30635.426733] 002: perf: interrupt took too long (6497 > 6461), lowering kernel.perf_event_max_sample_rate to 30750
```

As you can see, the messages are quite similar to those found in `dmesg`: in my opinion it's still worth to take a good look at `kern.log` after searching through `dmesg`, it can't hurt to try and you might find some more detailed informations.

### `syslog`

This file contains a different kind of logs, mostly userland related, that might shed some light on the misbehavior of applications. System control applications like to spam this log with mostly routine messages. You can search this log for events that happen in the userland, but be careful: there's *a lot* of noise in there. Here's an example:

```shell
$ cat syslog | tail -n 5
Nov 12 02:17:54 melchior whoopsie[1098]: [02:17:54] The default IPv4 route is: /org/freedesktop/NetworkManager/ActiveConnection/2
Nov 12 02:17:54 melchior whoopsie[1098]: [02:17:54] Not a paid data plan: /org/freedesktop/NetworkManager/ActiveConnection/2
Nov 12 02:17:54 melchior whoopsie[1098]: [02:17:54] Found usable connection: /org/freedesktop/NetworkManager/ActiveConnection/2
Nov 12 02:17:56 melchior whoopsie[1098]: [02:17:56] online
Nov 12 02:18:05 melchior systemd[1]: NetworkManager-dispatcher.service: Succeeded.
```

## Wrapping up

As always the Linux operating system offers advanced facilities for users, administrators and programmers, that allow for fine-grained tuning and control of the system. Looking at logs might not be a good way to stop the system from hanging in the first place, but it's a very useful tool to debug problematic situations.

## Sources

[https://www.plesk.com/blog/featured/linux-logs-explained/](https://www.plesk.com/blog/featured/linux-logs-explained/) makes a great attempt at explaining this same, very interesting, topic.
