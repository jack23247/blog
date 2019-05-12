---
title: "Installing AIX 4.3.3 on an IBM RS/6000 43P Model 100 (7248-100)"
date: 2019-05-12T23:13:00+01:00
categories:
 - sysadm
tags:
 - retrocomputing
 - unix
 - aix
 - ibm
---

![aix](assets/images/aix.png)

The IBM RS/6000 43P Model 100 (7248-100) is an entry-level workstation made by IBM in circa 1996. My machine has 96MB of RAM, 2x2GB SCSI disks and a 100MHz PowerPC processor. It came with AIX 4.1.5 and Windows NT 4.0 Workstation installed. The latest AIX revision it can run is 4.3.3, which is available on the net.

## Obtaining the OS images

AIX 4.3.3 has been preserved and installation images are downloadable from [WinWorld](https://winworldpc.com/product/aix/43x). For a minimal install you'll need only the first disk, but I suggest you also burn at least the **Bonus Pack CD 1**, which contains much needed manual pages.

## Booting the machine

The 7248-100 is a PReP machine, so it does not have the nice OpenFirmware ROM like CHRPs. To interact with the Firmware while booting you can press one of the function keys *before the last icon displays*. Please note that you should have an SMS Disk suitable for your machine if you intend to change the configuration and perform tests. Here is a summary of the boot options:



| Key  | SMS Disk Required | Function                     |
| ---- | ----------------- | ---------------------------- |
| `F1` | Yes               | Graphical System Maintenance |
| `F2` | No                | Select Boot Device           |
| `F4` | Yes (?)           | Text-Mode System Maintenance |
| `F5` | No                | Force Floppy Boot            |

> The 7248-100 can boot from a TFTPD server, you can use the `F2` menu to choose the parameters. 
>
> To boot Windows NT you need a special boot diskette.

Booting AIX is as simple as letting the computer boot or bringing up the boot menu with `F2` and selecting the CD-ROM as a boot device.

## Installing AIX

Installing AIX should be pretty straightforward: once the CD has booted, you'll be greeted with a message asking you to define the system console. Do as instructed. After a short while you'll be asked to choose the installation language, I decided to go with English. 

At this point you'll be in the Installation and Maintenance menu. I strongly advise against going with default configuration blindly, and I'd suggest that you at least review it once. Follow the on screen instructions and proceed.

I decided to do an Overwrite Install and to use both my disks (AIX installs itself on a logical volume which can span on multiple disks). I changed my locale settings, keyboard layout and decided not to install Trusted Computing Based (a system hardening layer).

Let the system installer work. Once it's done, the system will automagically reboot.

## Configuring the system

### First-time configuration assistant

Upon rebooting, the machine will prompt you with a dialog, expressing concerns about paging space. Basically, AIX is telling that the swap is too small and you need to change it. Luckily, the Configuration Assistant TaskGuide will assist you in various tasks, including that one. 

I decided to set the time, change root password, enlarge the swap and configure TCP/IP: the configuration is straightforward, except for choosing the hostname if you are using dhcp. For some reason you must either use a static network configuration or leave it as is.

Upon exiting the configuration assistant, you can either close it forever or reopen it on next startup. I'd choose the latter option at least for the first time, so you can easily tweak the system.

### Logging In

Like Solaris and HP-UX, AIX relies on Common Desktop Environment (CDE) for desktop usage. CDE is an antique but fun to use desktop system, with a very "Unix" feel to it. AIX's CDE makes things more user friendly by providing some desktop apps to automate system management. Feel free to explore CDE: it has a real nineties feel to it!

### Using SMIT

System Management and Information Tool, or `smit`, is an attempt by IBM to make AIX look more like its traditional menu-driven systems (OS/400, z/OS, etc.). Most system administration tasks can be brought out by using SMIT or its command-line counterpart `smitty`. 

To use SMIT you should start it up as root, either specifying nothing or a menu name. For example, assuming you're using the command line:

```shell
smitty          # brings up the main menu
smitty mktcpip  # brings up the TCP/IP configuration dialog 
```

> There is absolutely no difference in `smit` and `smitty`'s synopsis.

While `smit` uses Motif dialogs to select options, `smitty` emulates quite faithfully the OS/400's Fn-based menu navigation, and is the superior choice in my opinion: nevertheless, the use of  either of these tools is straightforward.

### Adding an user

After creating a root password you can login into the system and create a user by using SMIT. The user's password can then be changed via a classic `passwd`. You can also do this from the graphical tool.

### Extending logical volumes

This is a quite unnerving feature of AIX I was quite shocked about at first: the logical volumes have fixed, small sizes and will fill up almost immediately. It's up to the system administrator to check on those volumes and extend them as needed: I realized this while trying to `ftp` some files to my user's home directory and ended up with a dreadful "no space left on device" message.

A quick `df` revealed a grim looking situation:

```shell
# df -h
df: Not a recognized flag: h
Usage: df  [-P] | [-IMitv] [-k] [-s] [filesystem ...] [file ...]
# df
Filesystem    512-blocks      Free %Used    Iused %Iused Mounted on
/dev/hd4            8192      2760   67%      864    43% /
/dev/hd2          622592     44144   93%    12391    16% /usr
/dev/hd9var         8192      4520   45%      146    15% /var
/dev/hd3           24576     23384    5%       30     1% /tmp
/dev/hd1            8192      7840    5%       20     2% /home
#

```

1. `df` has no `h` (human-readable) flag 
1. Filesystem size is specified in *amount of 512-byte blocks*
1. `/usr` is almost full
1. `/home` is 4MB big

What the heck? How do I enlarge filesystems? Thanks to [this guy](http://geekswing.com/geek/how-to-expand-a-filesystem-in-aix-df-lsvg-chfs/) I was able to start poking around (I had not installed man pages yet). 

```shell
# lsvg -p rootvg
rootvg:
PV_NAME           PV STATE    TOTAL PPs   FREE PPs    FREE DISTRIBUTION
hdisk0            active      572         455         113..82..31..114..115
hdisk1            active      537         521         108..107..91..107..108
#
```

Ok, so, what does this mean? It means we have 1109 phisical partitions spanning on two phisical volumes, of which 976 are free. Logical partitions are made of phisical partitions (like an LVM volume is made of Extents) , so basically we have almost 90% of our disk space completely wasted. Yuck. Let's try to enlarge `/home` then:

```shell
# df /home
Filesystem    512-blocks      Free %Used    Iused %Iused Mounted on
/dev/hd1            8192      7840    5%       20     2% /home
# chfs -a size=512M /home
chfs: 0506-908 Cannot reduce size of file system.
#
```

Sorry, what? The partition is 4MB, which is NOT less than 512M... or is it?

Apparently, AIX 4.3.3 does not really check what you type after `size=`, because if you ask [this site](<https://sites.ualberta.ca/dept/chemeng/AIX-43/share/man/info/C/a_doc_lib/cmds/aixcmds1/chfs.htm>), it expects an *nbpi value*. Remember how `df` did not support human-readable format? Looks like this version was too early to do any math for you: you have to specify the size in *amount of 512-byte blocks*.

I pretty much screwed up my filesystems before understanding this simple fact.

```shell
# chfs -a size=16384 /home
Filesystem size changed to 16384
# df /home
Filesystem    512-blocks      Free %Used    Iused %Iused Mounted on
/dev/hd1           16384     15776    4%       20     1% /home
#
```

Bingo. `/home` is now... 8MB big! You can tweak pretty much everything this way.

> Only later I found out that you can use the (slow as heck) "Volumes" graphical tool to easily extend volumes. Yikes! 

### Adding System software

You can add system software via either SMIT or the toolbox application. System software spans Volume 1 through 4, yet I've not found many of those tools useful.

## Useful links

- [Collection of useful AIX commands](http://stromberg.dnsalias.org/~strombrg/Useful-AIX-commands.html)
- [AIX free software archive](http://www.bullfreeware.com/index2.php?page=lpp)

 ## To Sum Up

AIX is quite a strange *nix system, yet it's fun to use. The numerous differences make it a fun diversion even from classic SysV unices like Solaris and HP-UX

