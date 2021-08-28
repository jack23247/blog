---
title: "Installing Solaris 10 on a Sun Netra T1 from the network"
date: 2021-05-10T23:20:00+01:00
categories:
 - sysadm
tags:
 - server
 - laptop
 - unix
 - solaris
 - sun
 - netra
 - netboot
 - network
---

Since acquiring a Sun UltraSCSI disk array, I've decided to revive an old [Netra T1](https://dogemicrosystems.ca/pub/Sun/System_Handbook/Sun_syshbk_V4.1/Systems/Netra_T1_AC200/Netra_T1_AC200.html) (`aldebaran.local.net`) I had lying around to make an Unix media box. Since I'm a cheapskate and I didn't want to buy a blank DVD to install Solaris (and partly because the T1 does not have a DVD drive), I decided that netbooting would be the sanest option.

In 2021, a SPARC machine can be netbooted in different ways, namely:

1. Solaris JumpStart from a Solaris server
2. Solaris JumpStart equivalent from a Linux server
3. `sunboot`

The JumpStart process is a custom `tftpboot`+NFS+`bootparamd`+whatever streamlined solution offered by Sun for decades that can be easily set up on any Solaris machine, and an equivalent procedure should be easy to cobble up using a Linux server since all of the necessary servers are there (more on this later, though). On the other hand, `sunboot` is a novel approach that uses Vagrant and VirtualBox to set up an automated Debian VM that will do the heavy lifting for you. I've used `sunboot` to install Solaris 6 (SunOS 5.6) on my SPARCStation 5 in the past, but this time I decided that it would be nice to learn how to do this manually (also, I have no idea if `sunboot` does support Sol10).

Armed with my newfound netbooting knowledge, I decided that my laptop would be more than enough to netboot the T1 and started fiddling around with it.

> I swear I did not mean to turn this blog into a big netbooting series but it's so much fun, I want to try PXE (maybe even PXE on weird buggy BIOSes (InsydeH2O <s>(lisp-p?)</s>)) sometimes...

## Overconfidence

I decided to follow [this nice article]( https://jackwilsdon.me/solaris-10-jumpstart-linux/ ), as it described the procedure clearly: sadly, while I could reach `tftpboot` and download the kernel, it would then refuse to mount the root filesystem from NFS. This is an excerpt from one of several (failed) boot attempts with different parameters, [verbose options]( https://wiki.xdroop.com/space/Sun/Jumpstart/panic+-+boot:+Could+not+mount+filesystem) and stuff:

```
[...]
Netra T1 200 (UltraSPARC-IIe 500MHz), No Keyboard
OpenBoot 4.0, 2048 MB memory installed, Serial #51327197.
Ethernet address 0:3:ba:f:30:dd, Host ID: 830f30dd.



Executing last command: boot net:dhcp -v install
Boot device: /pci@1f,0/pci@1,1/network@c,1:dhcp  File and args: -v install
Timeout waiting for BOOTP/DHCP reply. Retrying ...
3a000 Using BOOTP/DHCP...
BOUND: IP address is: 10.0.0.124
Found 10.0.0.1 @ e8:6a:64:74:7b:87
BOOTP/DHCP configuration failed!
panic - boot: Could not mount filesystem.
Program terminated
ok
```

After a while I gave up, and started fiddling around with the server configuration. Being unfamiliar with the whole process, I decided that DHCP booting was too risky and that it would be safer to revert to a classic RARP setup.  This [older but still well-written article]( http://www.asgaur.com/wp/solaris-jumpstart-from-a-linux-server/  ) described such a procedure, using the `bootparamd` daemon to supply parameters to the client, thus rendering the DHCP server unnecessary. As you can see, this failed on a completely different level:

```
Netra T1 200 (UltraSPARC-IIe 500MHz), No Keyboard
OpenBoot 4.0, 2048 MB memory installed, Serial #51327197.
Ethernet address 0:3:ba:f:30:dd, Host ID: 830f30dd.



Executing last command: boot net -v
Boot device: /pci@1f,0/pci@1,1/network@c,1  File and args: -v
Timeout waiting for ARP/RARP packet
Timeout waiting for ARP/RARP packet
3a000 Using RARP/BOOTPARAMS...
Internet address is: 192.168.2.101
```

OpenBoot hangs because of... no obvious reason. After fiddling with wireshark for a while I realized that `rpc.bootparamd` wasn't answering the client's broadcast request at all! Yuck. This seems to be related to [an old bug](https://bugs.launchpad.net/ubuntu/+source/netkit-bootparamd/+bug/1610184) in the bootparamd package that's never been fixed, probably because this thing went out of fashion years ago.

> [This excerpt](https://docstore.mik.ua/orelly/networking_2ndEd/nfs/ch08_06.htm) from "Managing NFS and NIS, Second Edition" was instrumental in understanding the weirdness of `rpc.bootparamd`, even if I ultimately failed to solve the issue. 

At this point I kinda gave up, and the poor Netra became a fancy floormat for a while. 

## *Of dust bunnies and Compaq laptops*

A side effect of late spring in the countryside is the amount of dust bunnies that form due to all sorts of tree pollen littering the floors, and my study is no exception: the poor T1 was sitting there, completely covered in dust and pollen, and I immediately felt sorry for it. While I was tucking it away in a cleaner place, an idea struck me: what if I try to netboot it from Solaris? As you might (not) remember, I do in fact have [an x86 laptop running Solaris 10](https://jack23247.github.io/blog/sysadm/sol10-laptop-a/) (named `polaris.local.net`) lying around. 

> Keep in mind that you might need to enable unsafe hashing algorithms if you want to connect to your Solaris machine via SSH, see https://www.openssh.com/legacy.html

### Preparing the boot server

If there's one thing that Sun and Oracle can be praised for, it's <s>killing SPARC</s> the awesome documentation they continue to provide: while it might seem daunting at first, it's very thorough and can be followed quite literally. Also, they don't shuffle it around every few years like or remove it completely like HPE and IBM do... for now.

Now, following [this nice guide]( https://docs.oracle.com/cd/E26505_01/html/E28039/customjumpsample-5.html#scrolltoc) provided by Sun engineers, we are going to transform our humble laptop into a JumpStart server: what a catchy name!

1. Since the T1 has LOM over Serial, accessed using a Cisco console cable, I configured `tip` to use my laptop's serial port following [this guide](http://www.softpanorama.org/Solaris/Startup_and_shutdown/serial_console_on_solaris.shtml). Now we can use `tip` to interact with our client:

   ```
   polaris# tip hardwire
   
   lom>
   ```

   Hardware serial ports are handy! 

2. Time to mount the ISO, it's a bit more involved than on Linux but easy enough:

   ```
   polaris# lofiadm -a /home/quartz/Desktop/sol-10-u11-ga-sparc-dvd.iso 
   polaris# sudo mount -F hsfs -o ro /dev/lofi/1 /mnt
   polaris# ls /mnt
   boot       installer                         platform
   Copyright  Offer_to_Provide_Source_Code.txt  Solaris_10
   ```

   I think that having a tool like `lofiadm` is nice since the extra step shows you the separation between the (virtual) device and the filesystem that resides on it, and it helped me learn more about how loop devices work.

   > Having tucked the Solaris ISO on an old USB hard disk, I was afraid that it would've been impossible to mount. But to my surprise, JDS automagically mounted the ExFAT partition while I was skimming trough the manpages, so it was a matter of dragging and dropping!

3. Let's start with the guide: first of all, we need to create a folder for our NFS export and let the scripts on the install DVD copy the appropriate files over: 

   ```
   polaris# mkdir -p /export/install/sparc_10
   polaris# cd /mnt/Solaris_10/Tools/
   polaris# ./setup_install_server /export/install/sparc_10/
   Verifying target directory...
   Calculating the required disk space for the Solaris_10 product
   Calculating space required for the installation boot image
   Copying the CD image to disk...
   Copying Install Boot Image hierarchy...
   Copying /boot netboot hierarchy...
   Install Server setup complete
   polaris#
   ```

   At this point, the disk is not needed anymore and can be unmounted and ejected (or removed via `lofiadm`).

4. Now we are going to setup JumpStart itself: it's not a program, but more like a set of procedures and configuration files that can be copied over from the installation folder. Let's do just that now:
   ```
   polaris# mkdir /jumpstart
   polaris# cp -r /export/install/sparc_10/Solaris_10/Misc/jumpstart_sample/* /jumpstart
   ```
   Since the JumpStart folder needs to be accessible from the client, we need to share it over NFS: to do so, we need to add the following entry to `/etc/dfs/dfstab`:
   ```
   share -F nfs -o ro,anon=0 /jumpstart
   ```
   ...and tell the system to:
   ```
   polaris# shareall
   ```

5. JumpStart requires that we define a profile for our machine, and the matching between said machine and its profile is done by adding a rule to the aptly named `rules` file. The guide provides the following example:
   ```
   install_type  initial_install
   system_type   standalone
   partitioning  default
   cluster       SUNWCprog
   filesys       any 512 swap
   ```

   I felt that it was fine, and I saved it as `/jumpstart/aldebaran_prof`, but you might want to look at the docs and tweak it a bit.
   
   Now, we need to specify a rule to bind this file to our client: this can be accomplished by editing `/jumpstart/rules` and adding the following:
   ```
   #
   # Rule for Aldebaran (Netra T1)
   
   hostname aldebaran - aldebaran_prof -
   ```
   If you want a better understanding of what's going on, read the big comment at the top of the `rules` file: I had to play around with this a bit to make the client happy and it was very informative. 
   
   To finalize our changes, we need to run the `check` script:
   
   ```
   polaris# ./check 
   Validating rules...
   Validating profile host_class...
   Validating profile zfsrootsimple...
   Validating profile net924_sun4c...
   Validating profile upgrade...
   Validating profile x86-class...
   Validating profile any_machine...
   Validating profile aldebaran_prof...
   The custom JumpStart configuration is ok.
   polaris#
   ```
   
   ...which creates a `rules.ok` file that will be used by the client upon booting.
   
7. Last but not least, we need to run the `add_install_client` script from the install DVD, which will perform all sorts of black magic in our place. Honestly, I did not read the whole script but its output is quite verbose and it's very easy to follow if you've already set up a *nix netboot server. Oh, make sure that the host names and architecture are correct for your machine before issuing the command!
   ```
   polaris# cd /export/install/sparc_10/Solaris_10/Tools
   polaris# ./add_install_client -c polaris:/jumpstart aldebaran.local.net sun4u
   saving original /etc/dfs/dfstab in /etc/dfs/dfstab.orig
   Adding "share -F nfs -o ro,anon=0 /export/install/sparc_10" to /etc/dfs/dfstab
   making /tftpboot
   enabling tftp in /etc/inetd.conf
   Converting /etc/inetd.conf
   enabling network/rarp service
   enabling network/rpc/bootparams service
   updating /etc/bootparams
   copying boot file to /tftpboot/inetboot.SUN4U.Solaris_10-1
   polaris#
   ```
   ...and we're done! This was easier than I initially thought, and is a very nice solution that can be implemented with any old x86 junkbox you have lying around.
   
   > Friendly reminder: you must have your machine's host name, IP address and MAC address properly listed in `/etc/hosts` and `/etc/ethers`, or your client won't be able to boot!

### Booting the client

#### Loading the kernel

Now that our boot server is operational, we can boot our client machine in a fairly standard way. There are several steps in the boot process: first, when you issue the `boot net - install` command at the `ok` prompt, the client will attempt to download the kernel from the server. 

```
Netra T1 200 (UltraSPARC-IIe 500MHz), No Keyboard
OpenBoot 4.0, 2048 MB memory installed, Serial #51327197.
Ethernet address 0:3:ba:f:30:dd, Host ID: 830f30dd.




ok boot net - install
Boot device: /pci@1f,0/pci@1,1/network@c,1  File and args: - install
3a000
/
```

You will see many spinning cursors (in true Sun fashion) while booting. 

Once it's done, it will print the kernel version (similar to `uname -a`) and keep going:

```
SunOS Release 5.10 Version Generic_147147-26 64-bit
Copyright (c) 1983, 2013, Oracle and/or its affiliates. All rights reserved.SUNW,eri0 : 100 Mbps full duplex link up
Configuring devices.
Using RPC Bootparams for network configuration information.
Attempting to configure interface eri1...
Skipped interface eri1
Attempting to configure interface eri0...
Configured interface eri0
svc:/system/filesystem/local:default: WARNING: /usr/sbin/zfs mount -a failed: one or more file systems failed to mount
Setting up Java. Please wait...
Serial console, reverting to text install
Beginning system identification...
Searching for configuration file(s)...
Search complete.
Discovering additional network configuration...
```

It will take a while here, specifically while trying to autodetect the link on `eri1` which was not connected in my case: when it's done, you'll be presented with your standard Solaris "choose your destiny" prompt:


```
Select a Language

0. English
1. Brazilian Portuguese
2. French
3. German
4. Italian
5. Japanese
6. Korean
7. Simplified Chinese
8. Spanish
9. Swedish
10. Traditional Chinese

Please make a choice (0 - 10), or press h or ? for help:
```

And we're done! Just follow the on-screen prompts as usual: from this point, it's a completely standard Solaris installation procedure.

