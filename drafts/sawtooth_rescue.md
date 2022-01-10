# Sawtooth Rescue

Knowing well I care for old computers, one of my university professors gave me his old Sawtooth G4 (PowerMac 3,1) that's been "littering his office" for the last 15 years.

## Specs

The machine is quite standard, and has a ???MHz G4 7400 CPU, an ATI AGP video card, a SCSI card, a modem, the DL-DVD drive and almost nothing else. The RAM has been upgraded at some point, bumping it to about 192 Megabytes.

## Getting it to boot

When I first attempted to power it up, the system didn't display anything and there was no chime. Not knowing macs well, I was convinced that the PRAM battery had gone flat, and proceeded to remove that and perform a reset, to no avail. However, since the HDD was making some noise and the keyboard did respond to CapsLock and NumLock meant it was getting far enough to boot something even if it didn't display it, and that something else was amiss.

After leaving the machine for a couple weeks, a friend suggested I could try to clean the cards' contacts. Since the machine was quite dirty, I figured it would be a good idea, so I did clean and reseat everything I could on the mainboard except for the CPU daughterboard.

Sure enough, we started getting some output! It was quite garbled at first, but it turned out that it was just the DVI cable's fault, and switching to VGA solved the issue.

The system's still not chiming for some reason.

## Inspecting the HDD

The system has a dual boot setup that uses Yaboot to either load MacOS 9.2 or Mandriva Linux! The professor told me it was one of the first available Linux distributions for Mac.

## Recovering Passwords and Data

To attempt a password recovery, I decided to attach the HDD to my laptop with an IDE adapter: I was able to skim through the drive's contents and changed the root password on the Linux partition. For some reason the Mandriva

## Things to try:

### OpenFirmware Telnet (frpom macrumors)

```
" enet:telnet,1.2.3.4" io
```

Replacing 1.2.3.4 with a valid IP on your subnet, also make sure to include the space after the first quotation mark.

On the pc side: `telnet 1.2.3.4`.

### Attempt USB boot with OS9

There's an HDD image on WinWorld I can try to boot from

```
dev / ls
boot usb0/disk@1:5,\\:tbxi
```

OS9 won't boot from usb because it's R/W, I should patch the image. WHo cares I want OSX server anyway -> OSX server can't find the root on the usb key. Damn, try with vanilla osx next -> same stuff. Now OSX server complains about missing drivers? I'm confus.



Hdd imaged ok


## Considering an OS

The PowerMac3,1 runs a lot of stuff!

- MacOS Classic (9.2.1)
- MacOS X up to 10.4
- Old Linux (2.6.x) like Mandriva and Yellow Dog
- Adelie Linux (5.x!) musl-based distro
- MorphOS
