---
title: "Installing NetBSD on woefully underpowered hardware"
date: 2024-01-21T15:29:00+01:00
categories:
 - misc
tags:
 - netbsd
 - bsd
 - unix
 - retrocomputing
 - fun
---

It's no secret I have a knack for weird and generally useless computing devices. Today, I'll dive into the wonderful world of NetBSD, and how I installed the most ported OS of all time on those woefully underpowered gizmos. Oh, right, happy 2024!

<table style="width: 100%">
<tr>
<td><img src="https://raw.githubusercontent.com/jack23247/blog/master/img/raspberry-pi-logo-1.png" alt="rpi" style="height: 7em;" /></td>
<td><img src=".https://raw.githubusercontent.com/jack23247/blog/master/img/NetBSD.png" alt="netbsd" style="height: 7em;" /></td>
<td><img src="https://raw.githubusercontent.com/jack23247/blog/master/img/via_eden-1_6648_1.png" alt="eden" style="height: 7em;" /></td>
</tr>
</table>

## Raspberry Pi 1B

Story time: in my first year of high school (12 years ago!), I came across a magazine describing the Raspberry Pi and everything you could do with one. Not long after, I found an used Raspberry Pi 1B on eBay and promptly bought it. That, along with a Sun Fire V100 1U server I got from a fellow collector, <!-- BBK, I hope you're having fun with the ZX and the Dreamcast in the Afterlife :) --> was my first foray into Unix and Linux, which eventually led me to pursue a career in IT.

It's been a long while since I last used this Pi, which is astonishingly slow by today's standards! To circumvent the problem, i planned to run RISC OS on it, which is a simple cooperative multitasking OS that makes it feel _fast_. I like weird embedded OSes, but what I despise is the absurd business practices that revolve around it. The authors of RISC OS managed to open-source the OS code, but put the toolchain behind a paywall. 

That ended my brief foray into RISC OS, sadly. I would've loved to learn more about it but it wasn't meant to be. Maybe one day the toolkit will be released for free, and I'll go back to it.

Since I needed another lightweight OS, I turned to NetBSD. Here's how it went!

### Preparing the Pi

#### Disabling overscan

If you have black bars around your screen on the Raspberry Pi, just add `disable_overscan=1` to `/boot/config.txt`.

```
echo "disable_overscan=1" >> /boot/config.txt
```

#### Writing NetBSD to an SD card

Unlike on x86 systems, OSes for the Raspberry Pi come preinstalled in SD card images. Get the appropriate image for your Pi and `dd` it to an SD card. The system will resize the rootfs automatically based on the card's size when you boot for the first time.

### Configuring the System

The first time you boot the system from the SD card, nothing will work properly. We need to configure the system in order to use it.

#### Changing the hostname

To change the hostname, edit `/etc/rc.conf` and `:s/rpi/whatever`. If you want to change it without rebooting, you can use the `hostname whatever` command. You can also change the `/etc/motd` to your liking.

#### Changing the Timezone

```
# unlink /etc/localtime 
# ln -s /usr/share/zoneinfo/Europe/Rome /etc/localtime
```

#### Adding an user

To add an user, you can use `useradd`. Note that there won't be any `/home` directory initially, it will be created by `useradd` when you add your first user.

```
useradd -g users -G wheel -m username
passwd username
```

#### Setting up `pkgin`

Set-up `pkgsrc` to deliver binary packages via `pkgin`. The CDN seems to be broken at the moment, so I used the closest EU mirror.
```
# PKG_PATH="http://ftp.fr.NetBSD.org/pub/pkgsrc/packages"
# PKG_PATH="$PKG_PATH/$(uname)/$(uname -p)/$(uname -r | cut -d _ -f 1)/All"
# export PKG_PATH
# pkg_add pkgin
pkg_add: Warning: package `pkgin-23.8.1' was built for a platform:
pkg_add: NetBSD/earmv6hf 9.0 (pkg) vs. NetBSD/earmv6hf 9.3_STABLE (this host)
pkg_add: Warning: package `pkg_install-20211115' was built for a platform:
pkg_add: NetBSD/earmv6hf 9.0 (pkg) vs. NetBSD/earmv6hf 9.3_STABLE (this host)
pkgin-23.8.1: copying /usr/pkg/share/examples/pkgin/repositories.conf.example to /usr/pkg/etc/pkgin/repositories.conf
# echo $PKG_PATH >> /usr/pkg/etc/pkgin/repositories.conf
```

Then comment the line above what you added.

#### Using `pkgin`

`pkgin` is a convenient frontend for NetBSD's package system. It does what APT does for Debian, essentially. `pkgin` is very easy to use, but it won't spoil you like DNF does.

You can use `pkgin` to update the system as follows:

```
# pkgin update && pkgin upgrade
cleaning database from https://cdn.netbsd.org/pub/pkgsrc/packages/NetBSD/earmv6hf/9.0/All entries...
reading local summary...
processing local summary...
processing remote summary (http://ftp.fr.NetBSD.org/pub/pkgsrc/packages/NetBSD/earmv6hf/9.3/All)...
pkg_summary.bz2                               100% 3169KB  19.1KB/s   02:46    
calculating dependencies...done.
nothing to do.
```

You can also install new packages:

```
# pkgin in git-base doas vim
calculating dependencies...done.

11 packages to install:
  curl-8.4.0 doas-6.3p2nb1 git-base-2.42.0 libidn2-2.3.4 libunistring-1.1
  libxml2-2.10.4nb2 nghttp2-1.56.0 pcre2-10.42 vim-9.0.2122 vim-share-9.0.2122
  xmlcatmgr-2.2nb1

0 to remove, 0 to refresh, 0 to upgrade, 11 to install
36M to download, 97M of additional disk space will be used

proceed ? [Y/n] Y
[1/11] curl-8.4.0.tgz                         100% 1369KB   1.3MB/s   00:01    
[2/11] doas-6.3p2nb1.tgz                      100%   22KB  21.9KB/s   00:00    
[3/11] git-base-2.42.0.tgz                    100%   20MB   2.0MB/s   00:10    
[4/11] libidn2-2.3.4.tgz                      100%  313KB 313.3KB/s   00:00    
[5/11] libunistring-1.1.tgz                   100% 1630KB 815.0KB/s   00:02    
[6/11] libxml2-2.10.4nb2.tgz                  100% 1814KB 906.9KB/s   00:02    
[7/11] nghttp2-1.56.0.tgz                     100%  269KB 268.5KB/s   00:01    
[8/11] pcre2-10.42.tgz                        100% 1680KB 840.1KB/s   00:02    
[9/11] vim-9.0.2122.tgz                       100% 1714KB 571.5KB/s   00:03    
[10/11] vim-share-9.0.2122.tgz                100% 8290KB   1.6MB/s   00:05    
[11/11] xmlcatmgr-2.2nb1.tgz                  100%   29KB  29.2KB/s   00:01    
[1/11] installing xmlcatmgr-2.2nb1...
xmlcatmgr-2.2nb1: copying /usr/pkg/share/examples/xmlcatmgr/catalog.etc.sgml to /usr/pkg/etc/sgml/catalog
xmlcatmgr-2.2nb1: copying /usr/pkg/share/examples/xmlcatmgr/catalog.etc.xml to /usr/pkg/etc/xml/catalog
xmlcatmgr-2.2nb1: copying /usr/pkg/share/examples/xmlcatmgr/catalog.share.sgml to /usr/pkg/share/sgml/catalog
xmlcatmgr-2.2nb1: copying /usr/pkg/share/examples/xmlcatmgr/catalog.share.xml to /usr/pkg/share/xml/catalog
[2/11] installing libunistring-1.1...
[3/11] installing libxml2-2.10.4nb2...
[4/11] installing nghttp2-1.56.0...
[5/11] installing libidn2-2.3.4...
[6/11] installing pcre2-10.42...
[7/11] installing curl-8.4.0...
[8/11] installing vim-share-9.0.2122...
[9/11] installing git-base-2.42.0...
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/description to /usr/pkg/share/git-core/templates/description
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/hooks/applypatch-msg.sample to /usr/pkg/share/git-core/templates/hooks/applypatch-msg.sample
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/hooks/commit-msg.sample to /usr/pkg/share/git-core/templates/hooks/commit-msg.sample
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/hooks/post-update.sample to /usr/pkg/share/git-core/templates/hooks/post-update.sample
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/hooks/pre-applypatch.sample to /usr/pkg/share/git-core/templates/hooks/pre-applypatch.sample
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/hooks/pre-commit.sample to /usr/pkg/share/git-core/templates/hooks/pre-commit.sample
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/hooks/pre-rebase.sample to /usr/pkg/share/git-core/templates/hooks/pre-rebase.sample
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/hooks/prepare-commit-msg.sample to /usr/pkg/share/git-core/templates/hooks/prepare-commit-msg.sample
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/hooks/update.sample to /usr/pkg/share/git-core/templates/hooks/update.sample
git-base-2.42.0: copying /usr/pkg/share/examples/git/templates/info/exclude to /usr/pkg/share/git-core/templates/info/exclude
===========================================================================
$NetBSD: MESSAGE,v 1.3 2016/05/26 15:41:06 khorben Exp $

NOTE:   Pristine templates are located in:
        /usr/pkg/share/examples/git/templates.

To use the git-cvsimport repository conversion from CVS, install git-cvs.

To use the git-svn interface to Subversion, install git-svn.

===========================================================================
[10/11] installing doas-6.3p2nb1...
doas-6.3p2nb1: setting permissions on /usr/pkg/bin/doas (o=root, g=wheel, m=4511)
[11/11] installing vim-9.0.2122...
pkg_install warnings: 0, errors: 0
reading local summary...
processing local summary...
#
```

...and remove them along with their dependencies:

```
# pkgin rm neofetch vim 
Password:
2 packages to delete: 
  neofetch-7.1.0 vim-9.0.2122

proceed ? [Y/n] 
[1/2] removing neofetch-7.1.0...
[2/2] removing vim-9.0.2122...
pkg_install warnings: 0, errors: 0
reading local summary...
processing local summary...
# pkgin ar              
Password:
2 packages to be autoremoved:
  bash-5.2.15 vim-share-9.0.2122

proceed ? [Y/n] Y
[1/2] removing bash-5.2.15...
bash-5.2.15: removing /usr/pkg/bin/bash from /etc/shells
[2/2] removing vim-share-9.0.2122...
pkg_install warnings: 0, errors: 0
reading local summary...
processing local summary...
```

#### Configuring `doas`

`doas` is a `sudo` alternative that's supposedly easier to use and more secure. BSD systems in particular seem to prefer `doas` to `sudo`, in the same way Solaris preferred `pfexec`. You can use whatever you want, though.

Once you have installed `doas`, edit `/usr/pkg/etc/doas.conf` and make sure it contains the following:

```
# cat /usr/pkg/etc/doas.conf
permit :wheel
permit nopass :wheel cmd halt
permit nopass :wheel cmd reboot
```

This will let any user in the `wheel` group to execute commands as superuser, plus it will let them issue `halt` and `reboot` without typing their password.

### Using the system

The system feels quite responsive, but isn't blazingly fast. I still can't believe I used to run Deluge, a Samba server, and Apache2 off this thing!

#### Used space

The system fits nicely on a 4GB SD Card.

```
$ df -h
Filesystem         Size       Used      Avail %Cap Mounted on
/dev/ld0a          3.5G       1.2G       2.2G  34% /
/dev/ld0e           80M        17M        63M  21% /boot
kernfs             1.0K       1.0K         0B 100% /kern
ptyfs              1.0K       1.0K         0B 100% /dev/pts
procfs             8.0K       8.0K         0B 100% /proc
tmpfs              112M       8.0K       112M   0% /var/shm
```

#### Swag

Because, why not? 

<pre><span style="color:#A347BA"><b>                     `-/oshdmNMNdhyo+:-`</b></span>   <span style="color:#A347BA"><b>quartz</b></span>@<span style="color:#A347BA"><b>icecube</b></span> 
<b>y</b><span style="color:#A347BA"><b>/s+:-``    `.-:+oydNMMMMNhs/-``</b></span>           -------------- 
<b>-m+</b><span style="color:#A347BA"><b>NMMMMMMMMMMMMMMMMMMMNdhmNMMMmdhs+/-`</b></span>    <span style="color:#A347BA"><b>OS</b></span>: NetBSD 9.3_STABLE evbarm 
<span style="color:#A347BA"><b> </b></span><b>-m+</b><span style="color:#A347BA"><b>NMMMMMMMMMMMMMMMMMMMMmy+:`</b></span>             <span style="color:#A347BA"><b>Uptime</b></span>: 56 mins 
<span style="color:#A347BA"><b>  </b></span><b>-N/</b><span style="color:#A347BA"><b>dMMMMMMMMMMMMMMMds:`</b></span>                  <span style="color:#A347BA"><b>Packages</b></span>: 18 (pkg_info) 
<span style="color:#A347BA"><b>   </b></span><b>-N/</b><span style="color:#A347BA"><b>hMMMMMMMMMmho:`</b></span>                      <span style="color:#A347BA"><b>Shell</b></span>: ksh v5.2.14 99/07/13.2 
<span style="color:#A347BA"><b>    </b></span><b>-N/</b><span style="color:#A347BA"><b>-:/++/:.`</b></span>                           <span style="color:#A347BA"><b>Terminal</b></span>: /dev/pts/0 
<b>     :M+</b>                                   <span style="color:#A347BA"><b>CPU</b></span>: raspberrypi,model-b (1) 
<b>      :Mo</b>                                  <span style="color:#A347BA"><b>Memory</b></span>: 340MiB / 448MiB 
<b>       :Ms</b>
<b>        :Ms</b>                                <span style="background-color:#171421"><span style="color:#171421">   </span></span><span style="background-color:#C01C28"><span style="color:#C01C28">   </span></span><span style="background-color:#26A269"><span style="color:#26A269">   </span></span><span style="background-color:#A2734C"><span style="color:#A2734C">   </span></span><span style="background-color:#12488B"><span style="color:#12488B">   </span></span><span style="background-color:#A347BA"><span style="color:#A347BA">   </span></span><span style="background-color:#2AA1B3"><span style="color:#2AA1B3">   </span></span><span style="background-color:#D0CFCC"><span style="color:#D0CFCC">   </span></span>
<b>         :Ms</b>                               <span style="background-color:#5E5C64"><span style="color:#5E5C64">   </span></span><span style="background-color:#F66151"><span style="color:#F66151">   </span></span><span style="background-color:#33DA7A"><span style="color:#33DA7A">   </span></span><span style="background-color:#E9AD0C"><span style="color:#E9AD0C">   </span></span><span style="background-color:#2A7BDE"><span style="color:#2A7BDE">   </span></span><span style="background-color:#C061CB"><span style="color:#C061CB">   </span></span><span style="background-color:#33C7DE"><span style="color:#33C7DE">   </span></span><span style="background-color:#FFFFFF"><span style="color:#FFFFFF">   </span></span>
<b>          :Ms</b>
<b>           :Ms</b>
<b>            :Ms</b>
<b>             :Ms</b>
<b>              :Ms</b>
</pre>

#### Useful links

https://wiki.netbsd.org/ports/evbarm/raspberry_pi/
https://www.unitedbsd.com/d/6-netbsd-a-little-guide-for-newcomers
https://netbsd.org/docs/pkgsrc/using.html#using-pkg
https://wiki.netbsd.org/tutorials/how_to_set_up_per-user_timezones/

## IGEL D-210

If you combed through my blog, you might remember this little bugger from an old article I wrote detailing how I got Alpine running on this thing. It's an interesting system, basically an embedded platform with an x86 CPU and a weird BIOS. I even got Windows 2000 running on it!

### Set-up

I had to enable the Legacy USB Support option in the BIOS for it to see the USB drive. The system willl attempt to boot from Ventoy, but it will never find the rootfs. It might have worked if I dd-ed the appropriate image directly but I couldn't be bothered; time to cheat! I attached the CF card from the IGEL to my laptop with an USB to CF adapter and did the following:

```
$ sudo qemu-system-i386 -m 1G -drive format=raw,file=/dev/sdb -cdrom NetBSD-9.3-i386.iso -boot d
```

This let me install the system to the CF card using QEMU. I reduced the swap size to 128MB and installed only the Base System. Since I'm running a generic kernel and the installer essentially only copies the system to the destination, the system will boot just fine (unlike Windows, which might crash since the drivers get baked in the filesystem during the install process).

I proceeded to set up everything except networking (and, thus, no fancy `pkgin` setup) and shut down QEMU. I still enabled both `sshd` and `ntpdate` on boot so I couldn't forget to do so later on. 

I moved the CF card back to the IGEL and added the following to `/etc/rc.conf`:

```
hostname=IGEL
dhcpcd=YES
```

This let me start the DHCP client with `service dhcpcd start`, which gave me a valid IP address!

I then installed `pkgin` using the procedure described earlier. 

```
# export PKG_PATH="http://ftp.fr.NetBSD.org/pub/pkgsrc/packages/NetBSD/i386/9.3/All"
# pkg_add pkgin
pkg_add: Warning: package `pkgin-23.8.1' was built for a platform:
pkg_add: NetBSD/i386 9.0 (pkg) vs. NetBSD/i386 9.3 (this host)
pkg_add: Warning: package `pkg_install-20211115' was built for a platform:
pkg_add: NetBSD/i386 9.0 (pkg) vs. NetBSD/i386 9.3 (this host)
pkgin-23.8.1: copying /usr/pkg/share/examples/pkgin/repositories.conf.example to /usr/pkg/etc/pkgin/repositories.conf
# pkgin update && pkgin upgrade
processing remote summary (https://cdn.netbsd.org/pub/pkgsrc/packages/NetBSD/i386/9.0/All)...
pkg_summary.gz                                100% 5259KB  14.5KB/s   06:03    
calculating dependencies...done.

2 packages to upgrade:
  pkg_install-20211115nb1 pkgin-23.8.1nb2

0 to remove, 0 to refresh, 2 to upgrade, 0 to install
363K to download, 8B of disk space will be freed up

proceed ? [Y/n] 
[1/2] pkg_install-20211115nb1.tgz             100%  293KB 293.2KB/s   00:01    
[2/2] pkgin-23.8.1nb2.tgz                     100%   69KB  69.3KB/s   00:00    
[1/2] upgrading pkg_install-20211115nb1...
[2/2] upgrading pkgin-23.8.1nb2...
===========================================================================
The following directories are no longer being used by pkgin-23.8.1,
and they can be removed if no other packages are using them:

        /var/db/pkgin

===========================================================================
pkgin-23.8.1nb2: copying /usr/pkg/share/examples/pkgin/repositories.conf.example to /usr/pkg/etc/pkgin/repositories.conf
pkg_install warnings: 0, errors: 0
#
```
### Using the system

The system is fine, although it feels a bit cramped. It would probably benefit from a larger CF card. This little bugger has a VIA Eden CPU running at 500MHz, and it feels faster than the RPi. I must compare them!

```
# neofetch
                     `-/oshdmNMNdhyo+:-`   quartz@IGEL 
y/s+:-``    `.-:+oydNMMMMNhs/-``           ------------------ 
-m+NMMMMMMMMMMMMMMMMMMMNdhmNMMMmdhs+/-`    OS: NetBSD 9.3 i386 
 -m+NMMMMMMMMMMMMMMMMMMMMmy+:`             Uptime: 28 mins 
  -N/dMMMMMMMMMMMMMMMds:`                  Packages: 5 (pkg_info) 
   -N/hMMMMMMMMMmho:`                      Shell: sh 
    -N/-:/++/:.`                           Terminal: /dev/pts/0 
     :M+                                   CPU: IDT/VIA 686-class (1) 
      :Mo                                  Memory: 96MiB / 958MiB 
       :Ms
        :Ms                                                        
         :Ms                                                       
          :Ms
           :Ms
            :Ms
             :Ms
              :Ms

# uname -a
NetBSD IGEL 9.3 NetBSD 9.3 (GENERIC) #0: Thu Aug  4 15:30:37 UTC 2022  mkrepro@mkrepro.NetBSD.org:/usr/src/sys/arch/i386/compile/GENERIC i386
# cat /proc/cpuinfo 
processor       : 0
vendor_id       : CentaurHauls
cpu family      : 6
model           : 13
model name      : VIA Eden Processor  500MHz
stepping        : 0
cpu MHz         : 500.07
apicid          : 0
initial apicid  : 0
fdiv_bug        : no
fpu             : yes
fpu_exception   : yes
cpuid level     : 1
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge cmov pat clflush acpi mmx fxsr sse sse2 tm pbe nx rng rng_en ace ace_en ace2 ace2_en phe phe_en pmm pmm_en 
clflush size    : 64

# df -h
Filesystem         Size       Used      Avail %Cap Mounted on
/dev/wd0a          795M       225M       531M  29% /
tmpfs              240M         0B       240M   0% /tmp
kernfs             1.0K       1.0K         0B 100% /kern
ptyfs              1.0K       1.0K         0B 100% /dev/pts
procfs             4.0K       4.0K         0B 100% /proc
tmpfs              240M         0B       240M   0% /var/shm
#
```

## Benchmarks

I couldn't resist! NetBSD offers an implementation of LINPACK that can be used for benchmarking purposes. I also wanted to try 7-zip, but there are no prebuilt ARM binaries.

I performed the following tests:

1. Single precision LINPACK.
2. Double precision LINPACK.
3. Memory-to-Memory transfer of 1 million 512-byte packages from `/dev/zero` to `/dev/null`.
4. Disk transfer of 1 kilo 512-byte packages from `/dev/zero` to a file.

There is no scientific reason behind those benchmarks, so don't bother asking me why I chose them. The file transfer is kinda pointless, I think, and I got it from [here](https://linuxreviews.org/HOWTO_Test_Disk_I/O_Performance).

### Raspberry Pi

#### LINPACK Single Precision

```
$ linpacks
Rolled Single Precision Linpack

Rolled Single Precision Linpack

     norm. resid      resid           machep         x[0]-1        x[n-1]-1
       1.6        3.80277634e-05  1.19209290e-07 -1.38282776e-05 -7.51018524e-06
    times are reported for matrices of order   100
      dgefa      dgesl      total       kflops     unit      ratio
 times for array with leading dimension of  201
       0.01       0.00       0.01      47032       0.04       0.26
       0.01       0.00       0.01      45814       0.04       0.27
       0.01       0.00       0.01      50111       0.04       0.24
       0.01       0.00       0.01      46761       0.04       0.26
 times for array with leading dimension of 200
       0.01       0.00       0.01      49185       0.04       0.25
       0.01       0.00       0.01      46795       0.04       0.26
       0.01       0.00       0.01      49241       0.04       0.25
       0.01       0.00       0.01      51403       0.04       0.24
Rolled Single  Precision 46761 Kflops ; 10 Reps 
```

#### LINPACK Double Precision

```
$ linpackd
Rolled Double Precision Linpack

Rolled Double Precision Linpack

     norm. resid      resid           machep         x[0]-1        x[n-1]-1
       1.7        7.41628980e-14  2.22044605e-16 -1.49880108e-14 -1.89848137e-14
    times are reported for matrices of order   100
      dgefa      dgesl      total       kflops     unit      ratio
 times for array with leading dimension of  201
       0.02       0.00       0.02      35147       0.06       0.35
       0.02       0.00       0.02      35250       0.06       0.35
       0.02       0.00       0.02      35357       0.06       0.35
       0.02       0.00       0.02      35422       0.06       0.35
 times for array with leading dimension of 200
       0.02       0.00       0.02      35241       0.06       0.35
       0.02       0.00       0.02      35226       0.06       0.35
       0.02       0.00       0.02      35208       0.06       0.35
       0.02      -0.00       0.02      36737       0.05       0.33
Rolled Double  Precision 35422 Kflops ; 10 Reps 
```

#### Memory-to-Memory I/O

```
$ dd if=/dev/zero of=/dev/null bs=512 count=1000000 oflag=dsync
1000000+0 records in
1000000+0 records out
512000000 bytes transferred in 28.282 secs (18103387 bytes/sec)
```

#### File I/O


```
$ touch test.file
$ dd if=/dev/zero of=test.file bs=512 count=1000 oflag=dsync 
1000+0 records in
1000+0 records out
512000 bytes transferred in 13.918 secs (36786 bytes/sec)
```

### IGEL D-210

#### LINPACK Single Precision

```
$ linpacks
Rolled Single Precision Linpack

Rolled Single Precision Linpack

     norm. resid      resid           machep         x[0]-1        x[n-1]-1
       1.5        3.53703617e-05  1.19209290e-07  1.83582306e-05  1.33514404e-05
    times are reported for matrices of order   100
      dgefa      dgesl      total       kflops     unit      ratio
 times for array with leading dimension of  201
       0.01       0.00       0.01      57834       0.03       0.21
       0.01       0.00       0.01      57883       0.03       0.21
       0.01       0.00       0.01      57761       0.03       0.21
       0.01       0.00       0.01      57947       0.03       0.21
 times for array with leading dimension of 200
       0.01       0.00       0.01      57854       0.03       0.21
       0.01       0.00       0.01      57893       0.03       0.21
       0.01       0.00       0.01      57917       0.03       0.21
       0.01       0.00       0.01      57922       0.03       0.21
Rolled Single  Precision 57922 Kflops ; 10 Reps 
```

#### LINPACK Double Precision

```
$ linpackd
Rolled Double Precision Linpack

Rolled Double Precision Linpack

     norm. resid      resid           machep         x[0]-1        x[n-1]-1
       2.1        9.25510753e-14  2.22044605e-16 -8.68194405e-14 -7.20534743e-14
    times are reported for matrices of order   100
      dgefa      dgesl      total       kflops     unit      ratio
 times for array with leading dimension of  201
       0.01       0.00       0.01      57237       0.03       0.21
       0.01       0.00       0.01      57251       0.03       0.21
       0.01       0.00       0.01      57289       0.03       0.21
       0.01       0.00       0.01      57307       0.03       0.21
 times for array with leading dimension of 200
       0.01       0.00       0.01      57294       0.03       0.21
       0.01       0.00       0.01      57313       0.03       0.21
       0.01       0.00       0.01      57270       0.03       0.21
       0.01       0.00       0.01      57333       0.03       0.21
Rolled Double  Precision 57307 Kflops ; 10 Reps 
```

#### Memory-to-Memory I/O

```
$ dd if=/dev/zero of=/dev/null bs=512 count=1000000 oflag=dsync
1000000+0 records in
1000000+0 records out
512000000 bytes transferred in 13.410 secs (38180462 bytes/sec)

```

#### File I/O

```
$ touch test.file
$ dd if=/dev/zero of=test.file bs=512 count=1000 oflag=dsync 
1000+0 records in
1000+0 records out
512000 bytes transferred in 10.074 secs (50823 bytes/sec)
```

### Summary

|              | `linpacks` (Kflops) | `linpackd` (Kflops) | `dd of=/dev/null` 512x1M  (bytes/sec) | `dd of=test.file` 512x1K (bytes/sec) |
| ------------ | ------------------- | ------------------- | ------------------------------------- | ------------------------------------ |
| Raspberry Pi | 46761               | 35422               | 18103387                              | 36786                                |
| IGEL D-210   | 57922 (+19.3%)      | 57307  (+38.2%)     | 38180462  (+52.6%)                    | 50823 (+27.6%)                       |
|              |

The IGEL is quite a bit faster (34.4% on average) than the Raspberry Pi, but it is also beefier and it consumes more power. Overall, it feels snappier to use, but not by much.