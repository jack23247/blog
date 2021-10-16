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

>  Clear Linux as a desktop distribution is [dead](https://community.clearlinux.org/t/changes-coming-to-clear-linux-direction/4337). Long live Clear Linux as a cloud and server distribution.

I've been exploring Clear Linux as my daily driver laptop OS for quite a while, and I've found it pretty suited to what I need: it's modern, fast and has a small but nice community around it.

That said, it comes bundled with its own little quirks and problems: here are some examples of what I found annoying and had to fix.

## Uninstalling unnecessary software

Clear Linux uses *bundles* as its main packaging system, which are packages containing software A, B and C. Once you install a bundle, you can't choose to remove remove, for example, A or B from `swupd`: if you just wanted C, why didn't you install a bundle containing just that? Well, probably because it doesn't exist.

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

Now it should work just fine.

Because RPM is not a package manager, though, we need to find some way to update VS Code. We can leverage the flexibility of `rpm` to download and install the necessary package on-the-fly with a single command, such as this:

```shell
alias code-update='rpm -Uvh --nodeps https://update.code.visualstudio.com/latest/linux-rpm-x64/stable'
```

## Setting up Docker and/or Virt-Manager

Setting up these is quite straightforward, but `swupd` won't do everything for you like *APT&co.*

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

One of the most annoying things in CL is that it comes with almost no AV codecs installed (as they can't bundle ffmpeg for some licensing reason) and it's quite a bummer, as it's one of the distros where `vaapi` works best (at least on Intel Graphics).

I realized something was off when trying to watch a movie on Amazon Prime Video, which was constantly complaining about some "DRM error": I repeatedly tried to "fix" Firefox's Google DRM module.

Of course I ended up botching it and had to reinstall the module entirely. Since I could not understand anything wrong with it after reinstalling, I started looking elsewhere, and following [this post](https://community.clearlinux.org/t/how-to-h264-etc-support-for-firefox-including-ffmpeg-install/195) I was able to build and install `ffmpeg`, thus enabling system-wide video acceleration.

It turns out that the awful DRM module was not the culprit, as with `ffmpeg` codecs in place, I was able to watch videos normally.

Soon, though, another issue struck my not-so-peaceful movie evening: the video was jittering terrily when played in fullscreen, and was pratically unwatchable.

I traced the problem down to a nasty bug in Gnome's compositor, as described [here](https://bugzilla.mozilla.org/show_bug.cgi?id=1134077
) and [here](https://bugzilla.gnome.org/show_bug.cgi?id=741376#c15).

The fix described [here](https://github.com/ValveSoftware/steam-for-linux/issues/5866) stops the jittering and allows for fullscreen video playback without issues.

I'm not a Gnome power user (heck, I don't even like Gnome that much), so I wasn't able to permanently fix the issue. Guess it's time for another alias.

## Mitigating power consumption

> ***NOTE***<br>
> On my machine, with the optimized kernel, CL will consume about 5 to 7 watts of power in normal use, and about 4 in idle (as reported by powertop), with TLP enabled and configured and powertop in auto mode. Using thermald is NOT advised, and I might just switch to throttled  ([https://github.com/erpalma/throttled](https://github.com/erpalma/throttled)) when [this](https://github.com/clearlinux/distribution/issues/1337) is fixed.

If you have a ThinkPad (and other ultrabooks appearently), you should consider installing `throttled` to keep your silicon horses in check. I cooked up a list of prerequisites and steps to do so:

```shell
sudo swupd bundle-add c-basic python3-basic devpkg-cairo devpkg-glib devpkg-pygobject devpkg-gobject-introspection
git clone https://github.com/erpalma/lenovo-throttling-fix.git
sudo ./lenovo-throttling-fix/install.sh
```