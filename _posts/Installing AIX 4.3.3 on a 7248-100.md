# Installing AIX 4.3.3 on a 7248-100

The IBM RS/6000 43P Model 100 (7248-100) is an entry-level workstation made by IBM in circa 1996. My machine has 96MB of RAM, 2x2GB SCSI disks and a 100MHz PowerPC processor. It came with AIX 4.1.5 and Windows NT 4.0 Workstation installed. The latest AIX revision it can run is 4.3.3, which is available on the net.

## Obtaining the OS images

AIX 4.3.3 has been preserved and installation images are downloadable from [WinWorld](). For a minimal install you'll need only the first disk, but I suggest you burn at least the following:

- Volume 1 - Base OS
- Volume 2 - ?
- Volume 3 - 
- Volume 4 - 
- Bonus Pack CD 1 - 

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

### First-time configuration utility

### Using SMIT

System Management and Information Tool, or `smit`, is an attempt by IBM to make AIX look more like its traditional menu-driven systems (OS/400, z/OS, etc.). Most system administration tasks can be brought out by using SMIT or its command-line counterpart `smitty`. 

To use SMIT you should start it up as root, either specifying nothing or a menu name. For example, assuming you're using the command line:

```
smitty			# brings up the main menu
smitty mktcpip	# brings up the TCP/IP configuration dialog	
```

> There is absolutely no difference in `smit` and `smitty`'s synopsis.

While `smit` uses Motif dialogs to select options, `smitty` emulates quite faithfully the OS/400's Fn-based menu navigation, and is the superior choice in my opinion: nevertheless, the use of  either of these tools is straightforward.

### Adding an user

After creating a root password you can login into the system and create a user by using SMIT. The user's password can then be changed via a classic `passwd`.

### Extending logical volumes

This is a quite unnerving feature of AIX I was quite shocked about at first: the logical volumes have fixed, small sizes and will fill up almost immediately. It's up to the system administrator to check on those volumes and extend them as needed: I realized this while trying to `ftp` some files to my user's home directory and ended up with a dreadful "no space left on device" message.

A quick `df` revealed a grim looking situation:



### Adding System software

### Open Source tools

 ## To Sum Up

AIX is quite a strange *nix system, yet it's fun to use. The numerous differences make it a fun diversion even from classic SysV unices like Solaris and HP-UX

