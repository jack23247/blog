---
title: "Thin client tales: installing Alpine on an IGEL D210"
date: 2019-09-01T22:46:00+01:00
categories:
 - sysadm
tags:
 - linux
 - thin client
 - networking
 - updates
---

Hey blog, long time no see! Since I've started working things kind of got in the way: I have an article on *Rebuilding an AS400 ASP on an inprotected RAID-5 array without damaging it* underway but I've never been able to finish it. 

Good thing about work is that they were throwing away some old dusty boxes and I've been able to pickup a couple of things, including three IGEL D210 thin clients. These folks have [incredible specs](https://www.parkytowers.me.uk/thin/Igel/ud/ud2/D210/) and run Linux out of the box. After attempting to install various versions of Windows for *lulz*, I've settled on building a small Alpine Linux box I could use as a microserver. 

I'm using my Windows laptop to get internet access, as I use 4G in the apartment and my tethering-enabled router is still *en-route* from Amazon. That was terrible, I know.

As usual, **nothing** went smooth. I'll hide the most gruesome details for my own sake.

## Install notes

First of all, I took my trusted flat ethernet cable and connected the two boxes:

- `rigel`, the IGEL D210 running Alpine Linux
- `hydrogen4`, the ThinkPad T480 running Windows 10 1901 

I disabled all virtual interfaces on `h4` (I'll refer to the T480 this way from now on), I cleared the configuration on the Ethernet board and I enabled Internet Connection Sharing on the Wi-Fi adapter: Windows kindly prompted me that it was going to use `192.168.137.1` as the adapter's IP address. Noted.

Now it was time to start `rigel`. I prepped an USB key with Rufus and the latest Alpine ISO and I booted off it. To start the Alpine installer you have to login as `root` and then follow the on-screen instructions... if you want a headache: it looks like the installer is not setting the default route correctly, at least for my weird setup. `route add default gw 192.168.137.1` fixed it (ignore the `IOCTL` error).

Now it's time to run `setup-alpine`: I followed the on-screen procedure, using `192.168.137.11` as my IP, and happily hummed along.

> I've had an
> ```
> ssl_client: mirrors.alpinelinux.org: certificate verification failed: certificate is not yet valid error
> ```
> While downloading the install mirrors through HTTPS. This is a problem caused by a constant clock skew (`rigel` had a cheap CMOS battery that failed). To fix this, I had to restart `chrony`:
> ```shell
> service chronyd stop
> setup-alpine
> ```
> You should not have the problem if you don't exit alpine setup procedure after it has started `chrony`

I rebooted and re-ran the setup to clean everything up, just this time I hit `Ctrl-C` before the system actually started prepping the disk and explored a bit. Luckily, Alpine has nice [dedicated little tools](https://wiki.alpinelinux.org/wiki/Alpine_setup_scripts) to setup everything. I ran `setup-disk -s 0 -m sys` to see if disabling the swap partition could help. This time, the installation went smoothly and I was able to `reboot` into the newly installed system.

> Fun fact about the D210's default BIOS settings: CF's on the IDE0 Slave channel and... 32-bit transfers are **disabled**. It took me *hours* to understand why Win2k lagged as hell and my solid-state disk was acting like a Micropolis, so I thought I'd let you know. Just make sure you always check BIOS settings before randomly installing things.

## Post-install

All's good! Let's set up a user, install `sudo` to work more comfortably and connect through `ssh`...

```
# adduser -m /home/quartz quartz
# apk add sudo
# addgroup quartz wheel
```
Now, using `visudo`, uncomment line 82 (remove the `#` from `# %wheel ALL=(ALL) ALL`) to get users in the group wheel `sudo` access. Now we can say goodbye to our physical console:

```
C:\> ssh 192.168.137.11
```

And we're in! After some `/etc/profile` customization, let's see disk usage:

```
$ df -h
Filesystem                Size      Used Available Use% Mounted on
devtmpfs                 10.0M         0     10.0M   0% /dev
shm                     471.4M         0    471.4M   0% /dev/shm
/dev/sda2               842.8M    581.7M    201.5M  74% /
tmpfs                    94.3M    104.0K     94.2M   0% /run
df: /sys/kernel/debug/tracing: Permission denied
/dev/sda1                92.8M     17.5M     68.3M  20% /boot
```
![Not great, not terrible](https://raw.githubusercontent.com/jack23247/blog/master/assets/images/dyatlov.jpg)

Let's whip up some `apk` magic: 
```
$ apk list -I
syslinux-6.04_pre1-r3 x86 {syslinux} (GPL) [installed]
linux-firmware-av7110-20190322-r1 x86 {linux-firmware} (custom:multiple) [installed]
linux-firmware-20190322-r1 x86 {linux-firmware} (custom:multiple) [installed]
linux-firmware-rtl8192e-20190322-r1 x86 {linux-firmware} (custom:multiple) [installed]
linux-firmware-ene-ub6250-20190322-r1 x86 {linux-firmware} (custom:multiple) [installed]
linux-firmware-edgeport-20190322-r1 x86 {linux-firmware} (custom:multiple) [installed]
<...>
``` 
I've cut down on the list a bit: basically, this means that we have a lot of `linux-firmware` stuff installed. We don't need those packages, do we? Let's remove them at once:
```
$ sudo apk del linux-firmware-*
World updated, but the following packages are not removed due to:
  linux-firmware-av7110: linux-firmware linux-vanilla
  linux-firmware-rtl8192e: linux-firmware linux-vanilla
  linux-firmware-ene-ub6250: linux-firmware linux-vanilla
<...>
```

Nope, it won't do. [Looks like](https://unix.stackexchange.com/questions/475226/alpine-how-to-forcibly-remove-a-package-even-if-it-would-break-dependencies) we have to swap them with `linux-firmware-none` to do that:

```
$ sudo apk add linux-firmware-none
(1/85) Purging linux-firmware (20190322-r1)
<...>
(85/85) Installing linux-firmware-none (20190322-r1)
OK: 150 MiB in 54 packages
```

Good, let's run `df` now:
```
$ df -h
Filesystem                Size      Used Available Use% Mounted on
devtmpfs                 10.0M         0     10.0M   0% /dev
shm                     471.4M         0    471.4M   0% /dev/shm
/dev/sda2               842.8M    147.0M    636.2M  19% /
tmpfs                    94.3M    108.0K     94.2M   0% /run
df: /sys/kernel/debug/tracing: Permission denied
/dev/sda1                92.8M     17.5M     68.3M  20% /boot
```
As smooth as a rock! Now I have to think of a way to use our 600+M free space.
