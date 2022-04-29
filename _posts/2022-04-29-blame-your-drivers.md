---
title: "Bad GPU performance? Blame your drivers!"
date: 2022-04-29T18:08:00+02:00
categories:
 - misc
tags:
 - personal
 - gaming
 - nvidia
 - windows-xp
---

I have a little gaming system built around an ITX motherboard with an integrated nVidia GPU that I use for Windows XP retrogaming. The system is quite nice and, on paper, should enable me to enjoy a lot of classic PC-only games without adding a dime to the bill. The system is configured as follows:

#### Specs

- Zotac ION ITX with an  Atom N330, an nVidia ION chipset (GeForce 9400M), and 4GB of DDR2 RAM
- ThermalTake CORE V1 
- Kingston A400 240GB SSD
- Windows XP Service Pack 3 (w/USP4)

The system, as configured, draws at most 45W from the wall at all times, and the large intake fan of the Core V1 is more than enough to keep the chips cool.

## The Issue

Until today, I've had quite a few problems with the system, as it would often lock-up randomly (requiring an hard reset) and give me overall poor performance even on the least demanding games: Deus Ex was laggy and liked freezing randomly for a couple frames, Far Cry was unbearably slow and SimCity 4 was unplayable due to stuttering. I originally thought this was due to the Atom CPU, that is notoriously lacking in power due to a teeny weeny cache and the lack of instruction reordering.

## The Solution

Turns out that the system would crash inconsistently when trying to launch the nVidia Control panel. This, of course, was due to unstable and buggy drivers. Turns out 360.45 WHQL is **not adequate** for the ION chipset as it's too new, and it was likely never tested extensively with it. I rolled back to the last member of the 295 driver series, 296.10 WHQL, and the difference was immediately noticeable: the frozen frames and crashes are completely gone, and I was able to play through the remainder of Deus Ex in glorious 1080p with very stable framerates, dropping only in some areas due to fog effects I think are not accelerated by the chipset.

Similarly, FarCry and SimCity 4 are completely fixed: the former doesn't yield a stellar framerate but I can play it reasonably with high settings at low resolution. The only thing left, now, is to test it with more and more games, hoping it can deliver a good gaming experience.

## Goodies

If you have this board, here are some useful links:

#### Drivers

- Video: [360.45 WHQL](https://www.nvidia.com/Download/driverResults.aspx/77225/en-us) [296.10 WHQL](https://www.nvidia.com/download/driverResults.aspx/42438/en-us)
- Chipset (incl. Ethernet): [https://www.nvidia.com/Download/driverResults.aspx/14893/en-us](https://www.nvidia.com/Download/driverResults.aspx/14893/en-us)
- WiFi Card: [https://support.lenovo.com/us/en/downloads/ds007787-atheros-wireless-lan-driver-windows-vista-32-bit-64-bit-windows-xp](https://support.lenovo.com/us/en/downloads/ds007787-atheros-wireless-lan-driver-windows-vista-32-bit-64-bit-windows-xp)
- Realtek HD Audio: [https://support.lenovo.com/ro/en/downloads/ds015004-realtek-high-definition-audio-driver-for-windows-xp-32-bit-all-thinkcentre-system-thinkcenter-edge-71-edge-91-and-thinkstation-e20](https://support.lenovo.com/ro/en/downloads/ds015004-realtek-high-definition-audio-driver-for-windows-xp-32-bit-all-thinkcentre-system-thinkcenter-edge-71-edge-91-and-thinkstation-e20)

#### Unofficial Service Pack 4

- A video describing how to install USP4: [https://www.youtube.com/watch?v=vdIVMwV9MCk](https://www.youtube.com/watch?v=vdIVMwV9MCk)
- The files necessary to patch WSUS: [http://i430vx.net/files/wsusstuff/](http://i430vx.net/files/wsusstuff/)
- A curated list of software that runs on XP, including updaters: [https://skipster1337.github.io/posts/windows-software.html](https://skipster1337.github.io/posts/windows-software.html)



