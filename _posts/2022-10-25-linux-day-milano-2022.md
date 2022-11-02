---
title: "Highlights from Linux Day Milano 2022"
date: 2022-11-01T22:45:00+01:00
categories:
 - misc
tags:
 - linux
 - conference
 - linuxday
---

For the first time after the global pandemic we (UnixMiB) were able to organize another Linux Day and it's been an absolutely awesome experience. This year I decided not to bring a talk to the table and lean more onto the organization side instead. We had great participation, a lot of incredibly competent and talented people, and were able to build solid relationships. It was a really fun successful event that helped us renew interest in our LUG and in the Open Source movement, and we're all looking forward to next year's round.

Here are my personal highlights from the conference:

## PowerPC Progress Community -- PPC OSH Laptop update

> Speaker: Roberto Innocenti
>
> Slides: [Ecco fatto dopo 8 anni il Notebook Open Hardware - nonostante la guerra dei Chip](https://raw.githubusercontent.com/unixMiB/events/master/Linux%20Day%20Milano%202022/Ecco%20fatto%20dopo%208%20anni%20il%20Notebook%20Open%20Hardware%20-%20nonostante%20la%20guerra%20dei%20Chip%20-%20Roberto%20Innocenti%20-%20Power%20Progress%20Community.pdf)

The PPC progress community was back with their usual update on the progress of the PPC "Tyche" laptop board they're working on with the likes of AOpen and NXP and, boy oh boy, what an update!

They were able to finally lacate a supplier for the chassis (Slimbook) and get the PCBs designed and manufactured for prototyping. They ordered three prototypes and spent a staggering amount of money to actually populate the PCBs since jumping ahead of backorder for a small amount of parts costs a fortune because of ~~speculation~~ the semiconductor famine. Yuck. Funnily enough, the hardest part to locate was the HDMI connector, that had to be supplied by Slimbook just for the prototypes and will need to be changed in the future since it's basically unobtanium.

The good news is that we may be seeing the first fully functional prototypes in a laptop chassis as soon as the new heatpipes have been designed and manufactured.

The project has been able to chug along due to donations that were referred to by the speaker as "stupidly generous", and is a prime example of what the community can accomplish when there's a strong will to actually *do* something.

## Red Hat -- Contributing to the Linux Kernel is (kinda) easy

> Speakers: Rigel Di Scala and Carmelo Sarta
>
> Slides: [Come contribuire al kernel Linux](https://raw.githubusercontent.com/unixMiB/events/master/Linux%20Day%20Milano%202022/Come%20contribuire%20al%20kernel%20Linux%20-%20Rigel%20Di%20Scala%20e%20Carmelo%20Sarta.pdf)

A very interesting, if a bit brief, take on how to meaningfully contribute to the Linux kernel without getting shot in the back of the head by Linus and his goons (wink, wink).

The speakers explained the steps required to do so and demoed them live for the audience, a move they themselves declared temerary, and got in a rather heated argument with some of the audience who had prior experience with this kind of things. Awesome.

Sadly I had to perform my duties as a staff member and had to cut my permanence short due to the incessant amount of Q&A that was going on, but I can't wait to try and reproduce the steps on my own.

Moreover, the speakers briefly touched the subject of Rust kernel modules, showing how one can seamlessly integrate them with the Linux tree (or will be able to since Kernel 6.1). I honestly can't believe that the kernel has added Rust support before gccrs was ready, but I guess it may be a good thing since it will significantly reduce turnaround time as the groundwork has already been laid. Outtake: yay, LLVM kernel modules!

## Red Hat -- UEFI on a Raspberry Pi?!

> Speaker: Andrea Perotti

The ARM ecosystem has always been fragmented and, frankly, weird: most SBCs support only specific variants of the kernel, provided by the vendor, that are full of out-of-tree modifications and binary blobs.

This all changed when a consortium of vendors proposed a standard, mainly aimed at the datacenter, to bring all ARM devices closer together and avoid having to roll custom kernels to support each specific device tree.

This talk revolved around those standards, and showed us how an UEFI BIOS implementation targeting them was brought up by enthusiasts, who turned the Raspberry Pi 4 SBC into a fully fledged computer. The firmware blob must reside on the SD, but it enables you to boot from essentially any device (including iSCSI!!). This won't get rid of *all* binary blobs, but it's a start.

This is indeed a wonderful thing that will let you use an RPi (sadly only 4+) as if it was a "normal" computer, and boot any regular distro image that supports aarch64 out of the box. No weird trickery required except converting the image.
