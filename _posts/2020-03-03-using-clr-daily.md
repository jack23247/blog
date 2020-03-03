---
title: "Using Clear Linux as a daily driver"
date: 2020-03-03T03:57:00+01:00
categories:
 - linux
tags:
 - linux
 - configuration
 - thinkpad
 - modern
 - power
 - software
 - battery
---

I've been exploring Clear Linux as my daily driver laptop OS for quite a while, and I've found it pretty suited to what I need: it's modern, fast and has a small but nice community around it.

That said, it comes bundled with its own little quirks and problems: here is a small but growing collection of what I did to fix those.

## Uninstalling unnecessary software

Clear Linux uses bundles as its main package system: basically a bundle is a software package that contains A, B and C and, once you install it, you can't optionally remove A or B: if you just wanted C, you should've installed the C bundle after all.

This system works quite well, except for a single bundle: [`desktop-apps`](https://clearlinux.org/software/bundle/desktop-apps). This bundle installs along with it quite a long list of "useful" software, including the necessary File Roller and Gnome Libraries, but also Emacs, Evolution, Geary, Gedit...

To my dismay, I've found no way to remove those from the terminal, but I've found that, if you use the Gnome Software manager, you can remove the single apps you don't need.

## Using vanilla Visual Studio Code

Let me be clear on this one, I could've just installed a flatpak, but I want my system to be fully integrated: no sandboxing, no headaches.

Installing VS Code from Microsoft's website is fairly straightforward: get the RPM and use `rpm` to install (and update) it. 

```shell
sudo swupd bundle-add dnf
rpm -ivh --nodeps https://update.code.visualstudio.com/latest/linux-rpm-x64/stable
```

The `dnf` bundle contains the Fedora package manager to aid in the creation of bundles, which means it contains `rpm` as well, opening us to a whole world of standalone enterpri Code in a single command.

We should be done, but there is a nasty bug we need to fix before comfortably using Code: because fontconfig is not configured correctly, Code won't pick fonts up and everyfthing will look awful. The problem is, luckily, a single variable that is not exported correctly and it's an easy fix. Finding the root cause was a major pita though.

```shell
sudo echo "export FONTCONFIG_PATH=/usr/share/defaults/fonts" > /etc/profile
source /etc/profile
code
```

Now it should work just fine.se packages. Here we leverage the flexibility of `rpm` to download and install the necessary package on-the-fly, which means that we could do this:

```shell
alias code-update='rpm -ivh --nodeps https://update.code.visualstudio.com/latest/linux-rpm-x64/stable'
```

to let us update VS Code in a single command.

We should be done, except for a nasty bug we need to fix before comfortably using Code: since fontconfig is not configured correctly, Code won't pick fonts up and everyfthing will look awful. Luckily, it's enough to export a single variable to fix the issue, though finding the root cause was a major pita:

```shell
sudo echo "export FONTCONFIG_PATH=/usr/share/defaults/fonts" > /etc/profile
source /etc/profile
code
```

Now it should work just fine.

## Installing Telegram Desktop

Again, flatpak problem. There seems to be no way to download the Telegram package from their official website from the command line (damn those dynamic links). I got it from [https://telegram.org/dl/desktop/linux](https://telegram.org/dl/desktop/linux), and did the following:

```shell
sudo mkdir usr/local/bin
pushd ~/Downloads
tar -xvf tsetup.1.19.4.tar.xz && rm tsetup.1.19.4.tar.xz
sudo mv Telegram/Telegram usr/local/bin/telegram
sudo mv Telegram/Updater usr/local/bin/telegram-updater
rm -rf Telegram
popd
telegram
```

Please note that the `tsetup.1.19.4.tar.xz` filename is subject to change.

## Setting up Docker and/or Virt-Manager

Setting up these is quite straightforward, but `swupd` won't do everything for you like APT and its cousins. 

To install Docker:

```shell
sudo swupd bundle-add container-basic
sudo usermod -aG docker $USER
sudo systemctl enable --now docker
```

To install Virt-Manager:

```shell
sudo swupd bundle-add virt-manager-gui kvm-host
sudo systemctl enable --now libvirtd
```

## Enabling video playback

One of the most annoying things in CL is that it comes with almost no video codec installed (as they can't bundle ffmpeg for some licensing reason), while it's one of the distros where vaapi works best (at least on Intel Graphics).

I found out about this when trying to watch a movie on Amazon Prime Video, and I spent half an hour trying to "fix" Firefox's Google DRM module. Which was fine. Completely. 

I, then, started looking in other places, and following [this post](https://community.clearlinux.org/t/how-to-h264-etc-support-for-firefox-including-ffmpeg-install/195) I was able to build and install ffmpeg, thus enabling system-wide video acceleration. 

With ffmpeg in place, I was able to watch videos normally. After a couple minutes though, I noticed that the video was jittering terrily when played in fullscreen, and was pratically unwatchable. 

I traced the problem down to a nasty bug in Gnome's compositor, as described [here](https://bugzilla.mozilla.org/show_bug.cgi?id=1134077
) and [here](https://bugzilla.gnome.org/show_bug.cgi?id=741376#c15). 

The fix described [here](https://github.com/ValveSoftware/steam-for-linux/issues/5866) stops the jittering and allows for fullscreen video playback without issues.

## Mitigating power consumption

> **NOTICE** I couldn't solve this one yet: with the optimized kernel, CL will consume about 5 to 7 watts of power in normal use, and about 4 in idle (as reported by powertop), with TLP enabled and configured and powertop in auto mode. Using thermald is NOT advised, and I should really switch to throttled  ([https://github.com/erpalma/throttled](https://github.com/erpalma/throttled))when [https://github.com/clearlinux/distribution/issues/1337](https://github.com/clearlinux/distribution/issues/1337) is fixed.

### Installing throttled

> This is broken (as of Q32019)

If you have a ThinkPad (and other ultrabooks appearently), you should really consider the following:

```shell
sudo swupd bundle-add c-basic python3-basic devpkg-cairo devpkg-glib devpkg-pygobject devpkg-gobject-introspection 
git clone https://github.com/erpalma/lenovo-throttling-fix.git
sudo ./lenovo-throttling-fix/install.sh
```