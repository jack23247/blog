---
title: "OpenWRT on the Pirelli DRG-A226M"
date: 2024-12-10T10:10:30+0100 ## date +%Y-%m-%dT%H:%M:%S%z
categories:
 - misc
tags:
 - software
 - linux
 - hacking
 - openwrt
---

Did you know that Pirelli, besides making F1 cars spin on water, used to have a subsidiary that manufactured broadband networking equipment? Pirelli Broadband Solutions SpA (PBS) was a wholly owned subsidiary of the tire manufacturing company, which is headquartered here, just a few meters from our University's buildings. As far as I know, PBS went public in 2000, and by 2001 entered [a partnership agreement](https://www.fastweb.it/corporate/media/comunicati-stampa/pirelli-broadband-solutions-partner-tecnologico-di-fastweb-per-la-nuova-rete-a-banda-larga-next-generation-access-network/?lng=EN) with Fastweb for supplying ADSL and FTTH modem/router equipment to consumers. This led to some interesting Pirelli-branded devices which entered many people's homes in one way or another. Pirelli sold PBS to ADB Solutions a few years later, in 2010, when they were restructuring (possibly due to a certain disruptive economic event that happened in 2008).

Pirelli and ADB routers/modems were very popular in Italy thanks to those partnerships with ISPs, who insist on giving you a modem "for free" even if you tell them you already have one[^isps]; this results in people often accumulating old routers and boxes that they've no use for, like the DRG-A226M that is the protagonist of this article.

![drg](https://raw.githubusercontent.com/jack23247/blog/master/img/drg-a226m.webp)

[^isps]: This is for their own convenience, of course: since you have the right to change your ISP whenever you want, the providers charge you a very small fee for the router over a long period (usually a 24-month-minimum span) to keep you from changing, as you risk paying a steep fine if you recede early because "you're still paying for the router", and that constitutes a breach of contract... clever. 

All of those boxes have reached their end of life and are basically destined for the ~~landfill~~ RAEE recycling center, but the DRG-A226M holds some sentimental value for me[^modem], as it was my first broadband modem, so I decided to find out whether or not it could still be used for something.x

[^modem]: I still have fond memories of connecting to it and downloading GTA San Andreas mods from questionable websites on my Core 2 Duo laptop -- and, for a little while, it had a metered connection!

Several years ago I stumbled across [the DRG-A226M's page](https://oldwiki.archive.openwrt.org/toh/pirelli/drg_a226m) on the OpenWRT wiki and I decided to try and follow the steps: back then I was still quite green, but somehow I managed to crappily solder some wires to the pads for the serial port and install the OS on the router; I could never get it properly configured though.

A few weeks ago, while I was going through my stuff, the router resurfaced and I decided to see if I could really get it going this time. The [router's page on the modern OpenWRT wiki](https://openwrt.org/toh/pirelli/drg_a226m) is quite outdated but still holds valuable information, and another interesting resource is [this forum thread (:italy:)](https://www.ilpuntotecnico.com/forum/index.php?topic=76469.0).

## The Bootloader

To see the bootloader messages you need to attach an USB UART to the board: the wiki explains how to do so (heads-up: you'll need a soldering iron); I originally used some jumper wires, but I decided to swap them with leads made from capacitor legs instead, as they're much handier as they and allow you to remove the jumper wires when you're done. 

Once you've set up the connection, you'll need to attach to the board via a serial terminal emulator, such as PuTTY, at 115200 baud (8,n,1); if all went well, you'll see the following boot messages as soon as you plug in the power adapter:

```
CFE version 1.0.37-8.7 for NGRG 0002 BCM96358 (32bit,SP,BE)
Build Date: mer dic 17 10:22:13 CET 2008 (antonio@localhost)
Copyright (C) 2000-2005 Broadcom Corporation.

Boot Address 0xbe000000

Initializing Arena.
Initializing Devices.

Flash info:
device_id : ............227E
meminfo.nsect:..........128
totalSize:..............1000000
num_erase_blocks:...... 1
device_size:............1000000

Parallel flash device: name MX29L128DT, id 0x227e, size 16384KB

Resetting Switch....

CPU type 0x2A010: 300MHz, Bus: 133MHz, Ref: 64MHz
CPU running TP0
Total memory: 67108864 bytes (64MB)

Total memory used by CFE:  0x80401000 - 0x80528420 (1209376)
Initialized Data:          0x8041DE70 - 0x8041FC50 (7648)
BSS Area:                  0x8041FC50 - 0x80426420 (26576)
Local Heap:                0x80426420 - 0x80526420 (1048576)
Stack Area:                0x80526420 - 0x80528420 (8192)
Text (code) segment:       0x80401000 - 0x8041DE70 (118384)
Boot area (physical):      0x00529000 - 0x00569000
Relocation Factor:         I:00000000 - D:00000000

Resetting Secondary CPU...

Board IP address                  : 192.168.8.11:ffffff00
Host IP address                   : 192.168.8.182
Gateway IP address                : 192.168.8.1
Run from flash/host (f/h)         : f
Default host run file name        : vmlinux
Default host flash file name      : openrg.img
Boot delay (0-9 seconds)          : 9
Board Id (0-6)                    : DWV-S0
Number of MAC Addresses (1-32)    : 13
Base MAC Address                  : 02:10:18:01:00:01
PSI Size (1-64) KBytes            : 24
Main Thread Number [0|1]          : 0

*** Press any key to stop auto run (9 seconds) ***
Auto run second count down: 8
web info: Waiting for connection on socket 0.
CFE>
```

The `CFE>`[^cfe] prompt is where all the magic happens. Typing `help` will give us the following:

[^cfe]: I have no idea what "CFE" stands for.

```
CFE> help
Available commands:

w                   Write the whole image start from beginning of the flash
e                   Erase [n]vram or [a]ll flash except bootrom
r                   Run program from flash image or from host depend on [f/h] flag
p                   Print boot line and board parameter info
c                   Change booline parameters
f                   Write image to the flash
i                   Erase persistent storage data
b                   Change board parameters
reset               Reset the board
flashimage          Flashes a compressed image after the bootloader.
help                Obtain help for CFE commands

For more information about a command, enter 'help command-name'
*** command status = 0
```

Command `b` lets us mess with forbidden stuff, kinda like IPL C in AS/400 land; I do not recommend to mess with any of this stuff:

```
CFE> b
Press:  <enter> to use current value
        '-' to go previous parameter
        '.' to clear the current value
        'x' to exit this command
96358SV          ------- 0
96358VW          ------- 1
96358VW2         ------- 2
96358M           ------- 3
96358B           ------- 4
96358BGWE_OEM1   ------- 5
DWV-S0           ------- 6
Board Id (0-6)                    :  6
Number of MAC Addresses (1-32)    :  13
Base MAC Address                  :  02:10:18:01:00:01
PSI Size (1-64) KBytes            :  24
Main Thread Number [0|1]          :  0
*** command status = 0
```

Command `c`, on the other hand, lets us change the boot parameters, which is exactly what we need, as we'll flash the OpenWRT image via TFTP: you'll need to change the IP addresses for the Board (aka the Router), the Host (aka the TFTP server) and for the Gateway.

```
CFE> c
Press:  <enter> to use current value
        '-' to go previous parameter
        '.' to clear the current value
        'x' to exit this command
Board IP address                  :  192.168.8.11:ffffff00  192.168.1.50:ffffff00
Host IP address                   :  192.168.8.182  192.168.1.152
Gateway IP address                :  192.168.8.1  192.168.1.1
Run from flash/host (f/h)         :  f
Default host run file name        :  vmlinux
Default host flash file name      :  openrg.img
Boot delay (0-9 seconds)          :  9
*** command status = 0
web info: Waiting for connection on socket 1.
web info: Waiting for connection on socket 0.
```

Make sure that those addresses exist and are correct, and connect the router to another router using one of the ports on the back.

Now configure a TFTP server on the host[^tftp], download the [firmware image for OpenWRT 23.05.5 for the Pirelli DRG-A226M](https://downloads.openwrt.org/releases/23.05.5/targets/bcm63xx/smp/openwrt-23.05.5-bcm63xx-smp-pirelli_a226m-fwb-squashfs-cfe.bin), and rename it to something easy to type such as `firmware.bin`. At this point, you should be able to flash the router:

[^tftp]: You can do this [trivially on FreeBSD](https://wiki.freebsd.org/PXE%3ATFTPd%20Setup); on Linux it should be quite easy as well, and on Windows you can use something like [Tftpd64](https://pjo2.github.io/tftpd64/). Remember to disable your firewall or to add a rule to enable incoming connections to port 69/UDP.
 
```
CFE> flashimage firmware.bin
Loading 192.168.1.152:firmware.bin ...
Finished loading 5636100 bytes
SGP  tagVersion : 8
Flashing MAIN image at address: BE020000
signiture_2: IMAGE
remaining to flash .......  00000004



Resetting board...
```

At this point, the router should reboot into OpenWRT.

## OpenWRT

Welcome to OpenWRT! Here's a shortened version of my first boot log[^boot]:

```
CFE version 1.0.37-8.7 for NGRG 0002 BCM96358 (32bit,SP,BE)
Build Date: mer dic 17 10:22:13 CET 2008 (antonio@localhost)
Copyright (C) 2000-2005 Broadcom Corporation.

Boot Address 0xbe000000

<snip>

Closing network.
Starting program at 0x80a00000
[    0.000000] Linux version 5.15.162 (builder@buildhost) (mips-openwrt-linux-musl-gcc (OpenWrt GCC 12.3.0 r24012-d8dd03c46f) 12.3.0, GNU ld (GNU Binutils) 2.40.0) #0 SMP Mon Jul 15 22:14:18 2024
[    0.000000] Detected Broadcom 0x6358 CPU revision a1
[    0.000000] CPU frequency is 300 MHz
[    0.000000] 64MB of RAM installed
[    0.000000] board_bcm963xx: Boot address 0xbe000000
[    0.000000] board_bcm963xx: CFE version: 1.0.37-8.7
[    0.000000] printk: bootconsole [early0] enabled
<snip>
[   11.771584] procd: - init -
Please press Enter to activate this console.
```

[^boot]: If you want to see the full boot log, you can find it [here](https://raw.githubusercontent.com/jack23247/blog/master/misc/drg-owrt.log).

At this point, the system is running and, if you press enter a couple times, it should drop you into a root shell. Remember to set your password!

### Configuration

Now we can configure the router; on OpenWRT, all important configuration files live under `/etc/config`. We'll mess with the following:

- `/etc/config/system`, which defines system parameters and allows us to customize the behaviour of the LEDs;
- `/etc/config/network`, which allows us to configure the network (duh!).

#### System Configuration

Here's my system configuration. I cheated and did all of this through LuCI, but you can use it as a template if you want. 

```console
# cat /etc/config/system

config system
        option hostname 'pirellone'
        option timezone 'CET-1CEST,M3.5.0,M10.5.0/3'
        option ttylogin '0'
        option log_size '64'
        option urandom_seed '0'
        option description 'Pirelli DRG A226M-FWB'
        option zonename 'Europe/Rome'
        option log_proto 'udp'
        option conloglevel '8'
        option cronloglevel '5'

config timeserver 'ntp'
        list server 'time.inrim.it'

config led
        option sysfs 'green:VoIP'
        option trigger 'heartbeat'

config led
        option sysfs 'green:ethernet'
        option trigger 'netdev'
        option dev 'eth1'
        list mode 'link'
        option name 'Ethernet'
```


#### Network Configuration

Since there's no WAN port, we need to have two network configurations: one for connecting to the internet (e.g. if we want to download packages), and one for regular usage. 

The `network.client` config file shown here can be used to connect to the internet via any of the ports on the integrated switch, while the `network.router` file configures the board to act as a router and give out IP addresses to clients.

> I forgot to grab the DHCP server config off the router; you'll have to figure that out by yourself!

```console
# cat /etc/config/network.client

config interface 'loopback'
        option device 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fd96:a544:65fc::/48'

config interface 'lan'
        option type 'bridge'
        option device 'eth1'
        option proto 'static'
        option ipaddr '192.168.1.50'
        option netmask '255.255.255.0'
        option gateway '192.168.1.1'
        option dns '1.1.1.1'
        option nat '1'

# cat /etc/config/network.
network.client  network.router
# cat /etc/config/network.router

config interface 'loopback'
        option device 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fd96:a544:65fc::/48'

config interface 'lan'
        option type 'bridge'
        option device 'eth1'
        option proto 'static'
        option ipaddr '10.0.0.1'
        option netmask '255.255.255.0'

# rm /etc/config/network
# ln -s /etc/config/network.router /etc/config/network
# service network restart
[ 1675.341859] eth1: link DOWN
# [ 1686.318389] eth1: link UP - 100/full - flow control off
[ 1686.323849] IPv6: ADDRCONF(NETDEV_CHANGE): eth1: link becomes ready
```

### LuCI 

The Lua Configuration Interface (LuCI) is a pretty run-of-the-mill web interface that allows easy monitoring and management of your router without accessing the CLI. LuCI is particularly useful as OpenWRT is based on Linux and, to my knowledge, does not have a specific command line configuration tool a' la' Cisco IOS. To my surprise, LuCI was preinstalled and just worked:tm:; the Wiki states that you have to install it from the package manager, but that's probably not been true for a couple years.

### Wireless Networking

OpenWRT, being based on Linux, has pretty comprehensive wireless networking support and is documented [on their wiki](https://openwrt.org/docs/guide-user/network/wifi/wireless.overview). 

To get things going you need a couple of userland programs and drivers for your wireless card. Install the tools first as follows:

```console
# opkg update
# opkg install iw wireless-tools
```

This will give you access to commands such as `iw` and `iwconfig` that lets you configure wireless networking from the command line; at this point, we're "only" missing a suitable driver for our wireless chipset.

#### The Ralink Conondrum

This router/modem thing is supposed to support Wi-Fi networking through a mini-PCI[^mpci] card with a Ralink chipset which I was not able to get working. This was an unpleasant surprise as the Wiki states that it's enough to install some packages and _voilà_; I guess time fixes some things but breaks others.

[^mpci]: Not to be confused with the more ubiquitous mini-PCIe, more on this later.

At first, I wasn't even sure that my network card was working: there is no `lspci` by default in this OpenWRT build and, while I probably could get it from the repos, I believed there had to be another solution. 

Just looking at `dmesg`'s output didn't yield any obvious result, so I tried removing the card - no dice, I couldn't see the difference at first. I toyed around with the idea of `diff`ing the two logs, with and without the card, but it quickly became clear that a simple approach would not work due to timestamps. In the end, I managed to find [this script](https://gist.github.com/kbingham/c96812ed4e6c26f1c0264a022ab91a88) and got the following:

```console
$ bash dmesg-diff.bash dmesg0.txt dmesg1.txt
--- dmesg0.txt
+++ dmesg1.txt
@@ -53,10 +53,7 @@
  pci_bus 0000:00: root bus resource [mem 0x30000000-0x37ffffff]
  pci_bus 0000:00: root bus resource [io  0x8000000-0x8007fff]
  pci_bus 0000:00: No busn resource found for root bus, will use [bus 00-ff]
- pci 0000:00:01.0: [1814:0401] type 00 class 0x028000
- pci 0000:00:01.0: reg 0x10: [mem 0xffff8000-0xffffffff]
  pci_bus 0000:00: busn_res: [bus 00-ff] end is updated to 00
- pci 0000:00:01.0: BAR 0: assigned [mem 0x30000000-0x30007fff]
  PCI host bridge to bus 0000:01
  pci_bus 0000:01: root bus resource [mem 0x38000000-0x3fffffff]
  pci_bus 0000:01: root bus resource [io  0x8008000-0x800ffff]
@@ -181,7 +178,7 @@
  random: jshn: uninitialized urandom read (4 bytes read)
  eth1: link UP - 100/full - flow control off
  IPv6: ADDRCONF(NETDEV_CHANGE): eth1: link becomes ready
- jffs2: notice: (304) jffs2_build_xattr_subsystem: complete building xattr subsystem, 35 of xdatum (28 unchecked, 5 orphan) and 43 of xref (5 dead, 0 orphan) found.
+ jffs2: notice: (304) jffs2_build_xattr_subsystem: complete building xattr subsystem, 34 of xdatum (28 unchecked, 4 orphan) and 42 of xref (4 dead, 0 orphan) found.
  mount_root: switching to jffs2 overlay
  overlayfs: upper fs does not support tmpfile.
  urandom-seed: Seeding with /etc/urandom.seed
@@ -205,3 +202,5 @@
  urngd: v1.0.2 started.
  random: crng init done
  random: 28 urandom warning(s) missed due to ratelimiting
+ eth1: link UP - 100/full - flow control off
+ IPv6: ADDRCONF(NETDEV_CHANGE): eth1: link becomes ready
```

Aha! The answer was under my nose all along - by looking at the first chunk, we can see that a card with PCI Vendor ID `0x1814` and Device ID `0x0401` was detected; a quick search revealed that is indeed a Ralink RT2661T!

First mystery solved - now we need to hunt down a driver. Apparently the RT2661T should be supported by the `rt61pci` driver, which is not available anymore:

```console
root@pirellone:/etc/config# opkg list | grep rt61
rt61-pci-firmware - 20230804-1 - Ralink RT2561 firmware
```

Apparently, someone reported back in 2014 that the `rt61pci` driver was [not working correctly anymore](https://dev.archive.openwrt.org/ticket/18228), and the chipset was probably just too old and obscure for the devs to fix it.

A [newer driver](https://openwrt.org/packages/pkgdata/kmod-rt2x00-pci) called `rt2x00` is available in the repos and it can be installed:

```console
# opkg list | grep rt2x00
kmod-rt2x00-lib - 5.15.162+6.1.97-1-1 - Ralink Drivers for RT2x00 cards (LIB)
kmod-rt2x00-mmio - 5.15.162+6.1.97-1-1 - Ralink Drivers for RT2x00 cards (MMIO)
kmod-rt2x00-pci - 5.15.162+6.1.97-1-1 - Ralink Drivers for RT2x00 cards (PCI)
kmod-rt2x00-usb - 5.15.162+6.1.97-1-1 - Ralink Drivers for RT2x00 cards (USB)
# opkg install kmod-rt2x00-pci
Installing kmod-rt2x00-pci (5.15.162+6.1.97-1-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.4/targets/bcm63xx/smp/packages/kmod-rt2x00-pci_5.15.162%2b6.1.97-1-1_mips_mips32.ipk
Installing kmod-mac80211 (5.15.162+6.1.97-1-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.4/targets/bcm63xx/smp/packages/kmod-mac80211_5.15.162%2b6.1.97-1-1_mips_mips32.ipk
Installing kmod-rt2x00-lib (5.15.162+6.1.97-1-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.4/targets/bcm63xx/smp/packages/kmod-rt2x00-lib_5.15.162%2b6.1.97-1-1_mips_mips32.ipk
Installing kmod-rt2x00-mmio (5.15.162+6.1.97-1-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.4/targets/bcm63xx/smp/packages/kmod-rt2x00-mmio_5.15.162%2b6.1.97-1-1_mips_mips32.ipk
Configuring kmod-mac80211.
Configuring kmod-rt2x00-lib.
Configuring kmod-rt2x00-mmio.
Configuring kmod-rt2x00-pci.
[ 3844.892474] kmodloader: loading kernel modules from /etc/modules.d/*
[ 3844.955077] kmodloader: done loading kernel modules from /etc/modules.d/*
```

Even after installing the driver, though, the card won't show up in `wifi config` or `iwconfig`. I even confirmed that the driver had been, indeed, loaded correctly:

```console
# opkg files kmod-rt2x00-pci
Package kmod-rt2x00-pci (5.15.162+6.1.97-1-1) is installed on root and has the following files:
/etc/modules.d/rt2x00-pci
/lib/modules/5.15.162/rt2x00pci.ko
# lsmod | grep rt2x00pci
mac80211              591488  2 rt2x00pci,rt2x00lib
rt2x00lib              38160  1 rt2x00pci
rt2x00pci               1792  0
```

At this point, I called it a day; having already given this router a second life of sorts, I didn't feel like wasting more time on it. I tried looking online for replacement mini-PCI cards and I found some alternatives, such as the TP-Link TL-WN861N, but they're neither plentiful nor cheap enough for my tastes. I believe I could also just use an USB WiFi dongle, since the system is just running Linux.

## Outtakes

This fun little experiment proved that you can give new life to some of those old locked-down routers. Now, the question is: _should you invest time and effort into reflashing an old, slow, clunky router with weird hardware and little driver support?_

Probably not. But it's fun and educational!
