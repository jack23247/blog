---
title: "Defeating Windows' SAM"
date: 2022-06-07T00:10:00+02:00
categories:
 - misc
tags:
 - windows
 - security
 - sam
 - password
---

It is well known that Windows NT's Security Account Manager (SAM) wasn't all that sophisticated in the early days of NT, and that it can easily be circumvented. This can be useful if you're a retro-tech junkie as you'll often stumble across systems with unknown passwords, or forget which password did you use when you set up that old box.

## The SAM

The SAM is the Windows equivalent of `/etc/passwd`, and it's a database (technically a *Registry Hive*, or a file that stores a portion of the Registry) storing user names alongside password hashes. The SAM may be hardened using SysKey (or SAM Lock Tool), a program that encrypts it with an RC4 key to make it harder for an attacker to mess with it.

The idea behind SAM is that, by locking it early in the boot process, it won't be possible for an attacker to read or modify its contents... that is, until the computer has shut down. If you detach the hard disk from your PC or boot into another operating system that can access NTFS partitions, you'd be able to freely read and modify the contents of the SAM hive.

The easiest solution, at this point, is renaming the file: the next time it boots, the OS won't find the SAM file and it won't load any password information, making it possible for you to login with a blank password. Of course, this is what the SAM Lock Tool was created to prevent, but it looks like it may have been seldom used in the early days of NT.

## Bypassing SAM

I have attempted this on a QEMU VM by mounting the first partition of the raw disk image as a loop device and by renaming the SAM file. It can be done like this:

```
$ sudo mount -o loop,offset=32256 nt4.img /mnt 
$ mv /mnt/WINNT/system32/config/SAM /mnt/WINNT/system32/config/SAM.OLD
$ sudo umount /mnt 
```

The offset is needed because the NTFS partition does not start at zero, since quite a few sectors are reserved for the partition table and other vital informations. If you are using this trick on a physical HDD instead it's sufficient to attach it to a computer running Linux and mount the NTFS partition, then performing the same procedure.

## Using `chntpw`

`chntpw` is a tool that is able to read and modify the SAM file to remove the password hashes. Using `chntpw` is necessary in Windows XP because a check was introduced to mitigate this problem: by altering the SAM file without removing it altogether, this tool is able to clear the Administrator password without triggering the check.

## Conclusions

The security measures employed by early versions of Windows are not that sophisticated, and the attack vector is similar to manually editing the `/etc/passwd` file in *nix systems to blank the passwords and allow unrestricted access. This proves that, no matter the design of the system, the simplest things are often overlooked (maybe purposely so in some cases).
