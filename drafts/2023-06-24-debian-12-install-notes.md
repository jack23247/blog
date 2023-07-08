---
title: "Debian 12 Install Notes" 
date: 2023-06-24T14:51:25+0200 ## date +%Y-%m-%dT%H:%M:%S%z
categories:
 - sysadm
tags:
 - linux
---



## Installer

Download the netinstall image from debian's website, write it to an USB stick or boot it with Ventoy. Upon boot, choose _Advanced -> Expert Install_

Proceed with the default choices. At some point, the installer will ask you for an NTP Server. I like to use local servers, specifically `time.inrim.it`.

Proceed. To configure the disk, I usually have the installer automatically create a separate `/home` partitio, then change the types of both the root and home partitions to XFS. This can be done manually when you're presented with the summary, before committing the changes to disk.



Mirror: debian.mirror.garr.it
Use non-free firmware: yes
Use non-free software: yes
Enable source repos in APT: no
Check backported sw
Uncheck GNOME

## Userland

You'll have next to nothing on your machine right now. This is expected.

1. `usermod -aG sudo <Your Username>`
1. `vi /etc/default/grub`, add  `mitigations=off` to `GRUB_CMDLINE_LINUX_DEFAULT` and `GRUB_BACKGROUND=""`
1. `apt install gnome-core gnome-tweaks fonts-firacode lm-sensors`
1. `apt install htop nmon neofetch micro ftp telnet`
1. `apt install powertop tlp` 




[^strongarm]: Initially, support was limited to DEC/Intel StrongARM CPUs.

[^dreamcast]: CE also ran on the SEGA Dreamcast, and several games relied on it instead of running directly on the hardware. 
