---
title: "Core Transfer"
date: 2021-01-20T11:35:00+01:00
categories:
 - hardware
tags:
 - as400
 - system
 - os400
 - hardware
 - disk
 - core
 - ibm
---


>  "*To initiate a core transfer, please deposit substitute core in receptacle.*"
>
> ―[Aperture Science Announcement System](https://half-life.fandom.com/wiki/Aperture_Science_Announcement_System)

I've been collecting AS/400s for a couple years now, and I'm kinda sad that none of my articles are about those wonderful machines: I think I simply had nothing "useful" to say about them (which is not exactly correct, but whatever) until now.

You might also be wondering: "*what exactly is an AS/400?*". That's a simple question, yet hardly a simple answer. I'll write about it sometime.

> The really important thing is that, for once, I've found a bunch of really competent people that can actually correct me when I'm wrong and point me in the right direction, so **thank you IBM i Discord Community**, I owe y'all one.

Now, what do I have to say today about the AS/400s? Well, a couple things: for starters, my collection is about to grow a bit larger. Currently, I own the following machines:

| Model    | Name     | Description                                                 |
| -------- | -------- | ----------------------------------------------------------- |
| 9401-150 | The Good | A rather nice 150, my first 400                             |
| 9406-270 | The Bad  | A big, bad boy I got from a relative, it has a broken DASD  |
| 9401-150 | The Ugly | A 150 I've saved from someone's lawn. It's in a sorry state |

And I've just found a 9402, a late CISC machine, I'm about to pick up (COVID restrictions may delay that a bit). 

The star of today's show though, are the two 150s: you see, most 400s need a very specific kind of licensing information to work correctly. 

If you don't have the licenses for your Licensed Programs (as in, OS and extras) you're bound to reinstall everything every 70 days or so. Yuck. The older machines (CISC ones) go even further: every CISC CPU card has something called Vital Product Data (VPD), a bunch of numbers really, that is unique and tied to the actual system that's installed on disk. The CPU card VPD is factory paired to the front panel, which stores the pairing information on battery-backed RAM, and on the disk. The problem is: if the disk goes bad and the battery has depleted, well, you lose the pairing info and end up with a very heavy door stopper. Newer machines are somewhat more relaxed, but if you lose your LICKEYs you're bound to the 70 days cycle anyway.

> [2020-03-19:jack23247](mltjcp64+blog@gmail.com) Ok, so, I got the whole VPD thing wrong: it's more complicated than that and not limited to CISC hardware, so even, say, a 270 or a POWER4 iSeries might get fucked. Also, later machines (POWER3+ probably) have a dedicated bootstrap processor housed on a "VPD card" which kills the system if it goes bad. Nasty business.<br>Oh and don't quote me on any of this, seriously.

There is exactly one loophole in this whole licensing ordeal: the AS/400 9401 model 150. The 150 was a slow machine, yet aggressively priced for small business: it was so cheap because buying the machine would basically entitle you to run the whole V4R* distribution on it without getting any license. Moreover, it uses (almost) off-the-shelf peripherals, it's small, electrically cheap to operate, and does not weigh a ton: It's a hobbyist's dream machine for sure. 

After getting *The Ugly* from a friend about ten months ago, I've never really done anything with it besides finding out that it fails to IPL with an MFIOP error (I think the tape drive is electrically dead or something), but being unable to test it further I've always wondered what resided on the ASP (the disk pool). Afraid to kill *The Good* by fiddling around too much, I've never attempted to recover the disks and they basically sat there unused as "spares", just in case I'd need them. 

<figure>
  <img src="https://raw.githubusercontent.com/jack23247/blog/master/img/dummy_sys.jpg" alt="dummy_sys"/>
  <figcaption>Remember: this could happen if you IPL the wrong thing</figcaption>
</figure>

Today this changed: after having a talk with the aforementioned great guys on Discord I've decided to try a risky procedure: swap *The Good*'s good DASDs (let's call them "core" from now on, although it's definitely NOT IBM lingo) with *The Ugly*'s mysterious one. The concern was that my good core could be crippled as the MFIOP would "forget" the drive ordering and mix the disks up upon the next IPL.

> I think a little bit of explaining is due here:
>
> - Direct Attached Storage Device (DASD) is IBMese for "disk drive". So a single DASD is technically a disk drive, at least in the midrange world. I sometimes call DASD the whole disk array as that's closer to the original meaning of the term.
> - Attached Storage Pool (ASP) is a term that indicates how the OS "sees" (and partitions) a bunch of DASDs, think of it as an LVM or ZFS volume. An ASP can either be supported by an underlying RAID array (a RAID of DASD) or use the disks directly, just like LVM, and most of the time it encompasses a bunch of disks.
> - Initial Program Load (IPL) is the boot process. The firmware of the MultiFunctional I/O Processor (MFIOP), which is a fancy SCSI controller, looks for almost-SCSI (yes it's non-standard SCSI on most machines) devices, searching for a Load Source (the place where the LIC resides).
> - Licensed Internal Code (LIC) is, well... think of it as a kernel that includes an EFI loader, BIOS, and some kind of virtual machine (as in JVM, not a hypervisor). It's kinda convoluted: basically, everything down to firmware is stored on disk, so even to perform the most basic task you need a working Load Source.
>
> Since the Load Source is fundamental, and the ASP geometry data resides there, reordering the disks can be catastrophic, hence why we were concerned about the swap.

<figure>
  <img src="https://raw.githubusercontent.com/jack23247/blog/master/img/the_cores.jpg" alt="the_cores" style="zoom:50%;" />
  <figcaption>The cores</figcaption>
</figure>


I ignored the danger, knowing very well that the V4R5 installation on *The Good*'s core is extremely uninteresting and that I'm going to reinstall it afresh for sure someday (remember, media is enough on a 150), I decided to give it a go and swap the cores.

The installation procedure is quite risky: while the whole drive cage conveniently comes off, It's smashed two of my fingers against the rather small and pointy passive CPU heatsink, causing catastrophic bleeding (I literally smeared the heatsink with blood, it was quite a mess). I'm fine with it since the drive cage conveniently makes reordering drives impossible.

After getting a transfusion, I was hesitant to flick the switch: after doing so, there would be no turning back... At RAND_T minus 0, I hit the big white button, and the iconic  

```
01		B N
```

appeared on the display. Liftoff! It was time to see if the trick would work or mess up my perfectly good system. Yes, I forgot to go in Attended mode, but it really made no difference.

> `B` is the IPL side, `N` is the operating mode. Here's a chart:
>
> | A                                      | B                               | C                                                            | D                   |
> | -------------------------------------- | ------------------------------- | ------------------------------------------------------------ | ------------------- |
> | IPL the LIC's Permanent Copy (no PTFs) | IPL the current LIC and PTF set | Hardware Service Representative reserved IPL (aka black magic from Rochester) | IPL the service LIC |
>
> You commonly IPL from side B, and if you have any problems with the Program Temporary Fixes (PTFs, aka patches) applied to the system you may revert to A and apply a different set of patches. Side C is weird and undocumented, *hic sunt leones*, and side D is used to perform some maintenance described in the manuals.
>
> The operating modes are nothing special:
>
> | Manual (M)                                       | Normal (N)                   |
> | ------------------------------------------------ | ---------------------------- |
> | Attended IPL, aka interact with the boot process | Directly IPL to logon screen |
>
> You would normally use the Normal IPL (duh), but the attended IPL is very useful to work with the system as it allows you to go to the Dedicated Service Tool (DST), which is a set of tools used to configure the disks.

Or so I thought: the IPL proceeded at a snail's pace, as the system was probably quite confused about the serial numbers not matching up (or probably just needed to tidy up a bit since it was not IPLd since 2016). After grabbing a bite I came back and turned on the console, being greeted immediately with ~~Geiger counter~~ SCSI drive noises and a login prompt! 

**YES! The swap trick works!** I could (thankfully) login with the default `QSECOFR` password (which is `QSECOFR`, imagine if all Linux boxes came with `root`/`root`) and play around a bit. It seems like the machine had been used as an RPG box to port an older application from a System/36 to the integrated SSP environment on OS/400, while having a somewhat sketchy `192.168.0.1` IP assigned to it. 

<figure>
<img src="https://raw.githubusercontent.com/jack23247/blog/master/img/the_good_and_cores.jpg" alt="the_good_and_cores" style="zoom: 50%;" />
  <figcaption><i>The Good</i> with the cores</figcaption>
</figure>



After playing around with it for a while, I wanted to see if the swap trick would be reversible or not and, long story short, indeed it was! I can't state how refreshing it is to turn the machine I was afraid to boot because "I could damage drives" in a very flexible test lab for weird DASD configs.

And that's it. Here I stand, all excited because I found out that I can swap a disk with another (identical) one... This is why I find the AS/400s so interesting: everything is complicated in a way that makes it different than any other computer system you've used. While using a 400 you can basically take nothing for granted, and that eventually builds some solid skills.

