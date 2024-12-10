# FreeBSD 14 on the ThinkPad X220

https://www.c0ffee.net/blog/freebsd-on-a-laptop
https://genneko.github.io/playing-with-bsd/hardware/freebsd-on-thinkpad-t480/
https://docs.freebsd.org/en/books/handbook/parti/
https://www.stevekamerman.com/2008/07/freebsd-watch-command-is-not-gnulinux-watch-command/

## Installation

FBSD 14 boots from Ventoy 1.0.9.7

> [!Note]
> aha

### Network Connection

Bootonly -> needs inet 

Realtek RTL8188CE WiFi
FCC ID: ETSI/IT
IPV4/DHCP/No IPV6
DNS 1.1.1.1/1.0.0.1

Somewhat flakey, but it works. It won't acquire the DHCP lease at times, no idea why.

-or-

Intel 82579LM Ethernet (works fine)

### BIOS Bug Workaround

Yes

### Partitioning

Guided Root-on-ZFS should work even with a limited amount of RAM since we're not using any of the advanced features. 

Under **Pool Type/Disks** make sure to select **Stripe - No Redundancy** and add your drive (mine is named `ada0`).

### Mirror

The GeoDNS Main Site entry worked quite well for me (~750-1000KBPS using my phone's hotspot). 

### Config

- System Clock Set to UTC? YES
- Region Europe/Italy (8/23)
- Does CEST look reasonable? YES
- Services: sshd ntpd_sync_on_start powerd dumpdev

Add an user. Other groups: wheel video operator

When done, set up the Handbook. This will have the side effect of bootstrapping `pkg` for you so we don't have to do it later.

Done! Exit and reboot.

## Installing the GUI


### Installing the Video Driver

We need to install the i915 driver module, which is part of the `drm-kmod` metapackage. 

`pkg install drm-kmod`

We can try loading the module to see if it works as follows:

`kldload i915kms`

If everything went well, the console resolution should change and a bunch of messages should appear on the screen. To automatically load the `i915kms` driver module at boot time, we can use the `sysrc` command to automatically add an entry to `/etc/rc.conf`. 

`sysrc kld_list+=i915kms`

> In an attempt to use VAAPI (even if our iGPU probably won't support any useful codec in hardware) we can install the following packages: `libva-intel-driver mesa-libs mesa-dri`. I believe it's best to do this after you've installed everything else.

Now we can install X11 with `pkg install xorg`.

### Installing KDE and SDDM

https://docs.freebsd.org/en/books/handbook/desktop/

`pkg install plasma5-plasma konsole sddm`

```
sysrc dbus_enable="YES"
sysrc sddm_enable="YES"
sysrc sddm_lang="it_IT"

sysctl net.local.stream.recvspace=65536
sysctl net.local.stream.sendspace=65536
```

```
pkg install falkon fira firacode
```

If you want to control the brightness using the shortcuts you need to `kldload acpi_video` and/or add it to `kld_list`. Suspend/resume works ootb but it slow as fsck.

Do NOT install `xf86-video-intel`: it's horribly broken. Opt for DRM+modeset; the latter should be selected by default but you can modify the X config to use DPMS if you want. If you experience tearing with KWin, you need to go into the Compositor setting and select the lowest possible latency.

### doas

```
pkg install doas
echo "permit persist :wheel as root" > /usr/local/etc/doas.conf
```

You can try it with `doas whoami`.

### Wayland

Apparently, Wayland should work as well now. I couldn't figure out how to make it work, however.

