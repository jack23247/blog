---
title: "The bittersweet tale of the 9406-170"
date: 2021-04-01T13:35:00+01:00
categories:
 - hardware
tags:
 - as400
 - system
 - os400
 - hardware
 - disk
 - ibm
 - expansion
 - psu
---

I recently recovered an AS/400 9406-170 with an expansion chassis I got for scrap value: the seller advertised it as "barely functioning, can't find VGA port", but since it is quite loaded I decided to at least try and save the disks. 

When I first plugged it in, it turned out that Expansion Unit's PSU was completely dead: when the EU Planar was plugged into the BU Planar, the system would not IPL, complaining about a power failure, and the EU PSU would draw no amps from the wall, a clear sign that something was wrong. I decided to take a look around and see what I could do about it: it turned out that the SRC was related to the fact that the system could not communicate with the EU PSU through the EU planar. I thought that replacing the fuse would fix it, so I did, and tried plugging it in again: there was furious arcing and the fuse melted.

Time to find a workaround, I guess...

## Disabling the expansion unit

### Rearranging the system

Since the 170 can work in a standalone configuration, I looked at the System Builder to see if there was any viable option: it looks like the client bought this just for the disks, as the only card plugged in the EU planar was a #2838 LAN IOA that is definitely not necessary to IPL, and could be relocated anyway as the Base Unit #2290 planar supported at least four cards on its own, and wasn't fully loaded. I relocated the cards as follows:

| Feature Code | Card                      | Previous Location | New Location | In use?     |
| ------------ | ------------------------- | ----------------- | ------------ | ----------- |
| #2809        | LAN/WAN/Workstation IOP   | E10               |              | No          |
| #2838        | LAN IOA                   | E08               | C03          | Yes         |
| #2745        | WAN IOA                   | C08               |              | No          |
|              | 9401-150 WAN/Twinax MFIOA |                   | C09          | Yes         |
| #2740        | PCI RAID                  | C07               | C07          | Temporarily |

I also removed the whole EU Planar and ribbon cable, so I would not risk damaging them.

> ***NOTE TO READER***
>
> The MFIOA from the sad 150 is technically not supported, but I decided to try to use it anyway and (to my surprise) it works just fine.

### Powering the drives

Since the EU's PSU is toast, my plan was to use a common ATX psu to power the drive bays. I used a 350W supply from the spares bin and tested it by dry-running it with a jumper between the green cable and one of the grounds: luckily, it worked. The drives won't actually spin up until the controller tells them to, which is rather convenient, but shutdown needs to be done manually by removing the jumper.

## Attempting to IPL

 The first IPL attempt yielded SRC `675A9050` which, according to Problem Analysis 170 is: 

> Required cache data cannot be located for a disk unit. Perform “SDIOP-PIP23” on page 326.

A rough translation from IBMese to english would be: *the batteries on the RAID card are dead, you're done for!* 
And you know what? This is great! We know that the battery went dry a long time ago, so it means that the system is at least trying to boot off the RAID array. As instructed, I tried procedure SDIOP-PIP23 but I wasn't successful: the disk configuration has been botched enough for the chonkster to be unhappy. 

### Side D to the rescue

![](https://raw.githubusercontent.com/jack23247/blog/master/img/170_loading_slic_dst.jpg)


I decided to attempt a manual IPL from side D, and I got `B1014507` (load source not ready): this, again, was expected as the system was trying to IPL from the SLIC CD. The problem is that, even after inserting V4R4 I_BASE_01, it would repeatedly fail: the CD-ROM drive was dirty enough that it had trouble reading the disk! After a good cleaning, all was well and I could IPL to DST, and perform an IOP Data Cache Reclaim operation. 

This reclaim basically tells the system to discard everything that was in cache: a few objects might be lost or damaged, but it should not be bad enough to render the system unusable, which is the bare minimum we need to save the precious LICKEYs from certain doom.

## ABEND

Sadly, at this point things took a turn for the worse: all was well, the system re-IPL'd and started resyncing the RAID array and rebuilding the object database. This meant a sizable power draw, with both heavy disk activity and high CPU load. At this point, the PSU gave in and tripped. No SRCs, no fault light: it just went dark for a fraction of a second, bringing the whole system to a sudden halt. 

And this is the end of the story, for now: I have not confirmed what went awry in the PSU because it's riveted and not supposed to be opened for repairs at all, so I can either scour eBay for a cheap and (hopefully) working parts bin PSU or decide to open it up and attempt to repair it. 

Thus, I hope this article is only "part one" of a series where I'm able to put the system back online. 
