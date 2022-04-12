# Installing OS/2 Warp 3 Connect on an IBM PS/ValuePoint 433DX

*Half an OS for half a Computer... and of course I have the two halves that don't go together.*

Happy 2021 everyone! I just realized I forgot about that in the earlier post, so here you have it.

I realized, talking to someone from the AS/400 user group I coaxed into writing their blog, that I should focus more on *what* I do instead of *why* I do it, so I'm limiting all of the backstories to a paragraph or so.

## The computer

The PS/ValuePoint 433DX is a 486-class machine made by IBM. It's a fine motherboard, sporting PS/2, an onboard graphics card (Tseng Labs ET4000), an IDE controller, and a Pentium OverDrive socket. I traded this machine for my old Olivetti M6 420 Suprema with a local collector last year, and it came in a pretty standard configuration: 33MHz 486DX with 4MB of RAM, a missing ISA slot cover, an ugly CD-ROM drive, and a dead HDD. I replaced the HDD with an 850MB Western Digital Caviar, and the CD-ROM drive with a nice, albeit not period-correct, Yamaha unit. Ultimately, I added a network card based on the National Semiconductor AT/LANTIC DP83905 (NE2000-compatible), which sports both ethernet and AUI.

I've had some "problems" with this machine, but they were all my fault: the OS/2 installer would completely hang, pointing to some kind of memory-related error.

Since the machine would work fine with DOS, I decided to try and expand the memory to 8MB: hastily, I bought the wrong kind, costing me 10€ and almost a month, then the post office lost the package with the correct sticks, costing me another month and a half.
Thankfully the guys from the shop (Electromyne on eBay, in case you're wondering) have been hugely helpful and agreed to send me another kit for free with a safer (tracked) shipping method. It arrived a couple days later, and it turned out I was right: the RAM works fine and I was just short on it.

Long story short: if you need RAM for this system (6384-M50, the 433DX/Si is a different story and it requires DIMMs) get a kit of four FPM 30-pin unbuffered SIMMs with 70ns timing and parity.

## Installing OS/2

OS/2 Warp 3 Connect, which has been archived, is a kit comprising:
- two floppies to load the kernel and the installer
- a CD-ROM containing the distribution
- a BONUS PAK CD-ROM containing some extras

Upon booting, the system will:

1. load the OS/2 kernel from the first diskette,
2. ask you for the second diskette (which IBM calls `Disk 1`),
3. load the installer from the second diskette.

After this (painfully long) process, the actual installation procedure should start, warning you with a beep and ask you if you want to perform a "Simple" or "Advanced" install. Choose the latter, as it will let you change rather important stuff.

> (this IBM thingy beeps a lot, unlike its more famous competitor, and I think it's just great)

The first thing you'll be asked about is disk partitioning, and it's a tricky one: I think OS/2 makes BIOS calls to detect the HDD, or at least is very limited, as the old 800MB DOS partition on the Caviar would kill the installer (reporting an `FDISK` error) and drop me to the `C:\>` prompt. From there I ran `FDISK` manually and was able to delete the offending partition and reboot. Be wary that OS/2's `FDISK` is kinda different than the classic DOS one we're all accustomed to, and it has a more refined look. As all things IBM the help system is very good, so refer to that if you need to.

Upon rebooting I let the wizard do its thing to the disk and ended up with a 450-ish Megabyte partition: it's not like I'm going to use them all but it kinda pisses me off. Whatever, let's proceed: the system will let you format the partition, choosing either HPFS (recommended) or FAT and will copy over the installer files, then it will ask you to eject the floppy and it will reboot the machine once again.

The system will now boot from the hard drive and present you with the Hardware Configuration menu: this is a crucial step, as OS/2 is very temperamental. In my case, most things were configured correctly, but I had to change the CD-ROM Device Support from "OTHER" to "Non-listed IDE CD-ROM" or it would refuse to let you use the drive later on. You'll be asked about a printer, which I omitted since I don't have, and to confirm your Display Driver.

After confirming, the installer will ask you about which components you'd like to install, and if you'd like for it to search for existing Win3.11/DOS programs (more on that later). After some disk activity you'll be asked if you want to install the OS/2 Warp Connect networking components: if you don't have working drivers at hand, skip this. After copying the files, the system will attempt to autodetect the monitor you're using. Go for whatever resolution suits you best and continue.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk5OTI1Mzc3MF19
-->