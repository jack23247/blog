# Installing SCO UnixWare 7.1.4 in qemu

Okay, this one's a quickie. I have taken the habit of writing down install notes because I hate having to relearn this stuff over and over again, so here we are.

## Rationale

I like weird Unix[^unices] Systems and, sometimes, it seems like I enjoy ~~hurting~~ challenging myself.

UnixWare is a weird beast. It's one of the longest standing, traditional SVR Unix systems and it was essentially born on x86. UnixWare and its sibling OpenServer have passed hands several times, being caught in, buyouts, bankrupcies and the idiotic SCO vs The World[^kakyoin] lawsuits. Nowadays, they are probably the only surviving proprietary Unix distribution, along with AIX.

I have no idea what those systems were and are used for, besided them being [behind Pizza Hut's PizzaNet](https://www.santacruztechbeat.com/2016/09/15/back-sco-pizza-hut-made-headlines-pizzanet/), the first system of its kind. Nowadays, they are a funny relic from another era of computing, and they are a solid choice for older computers you want to run *nix on.

[^unices] What's the correct plural form of Unix? Unices? Does it even have one? And how is it supposed to be capitalized? UNIX?

[^kakyoin] And we all know how that went.

## Set-up

Here's the configuration I used for setting up the VM:

```
$ cat bringup.sh
qemu-img create -f qcow2 hd0.qcow2 8G
export CD_ROM_IMAGE uw714.CD1.Jun2008.iso
export CDR_OR_HD0 d # d->CD-ROM, c->HD0
qemu-system-i386 \
	-m 1G \
	-machine pc-i440fx-6.2,accel=kvm \
	-device lsi53c895a,id=scsi0 \
	-device scsi-hd,drive=drive0,bus=scsi0.0,channel=0,scsi-id=0,lun=0 \
	-device scsi-cd,drive=drive1,bus=scsi0.0,channel=0,scsi-id=1,lun=0 \
	-drive file=hd0.qcow2,if=none,id=drive0 \
	-drive file=$CD_ROM_IMAGE,if=none,id=drive1,format=raw \
	-boot $CDR_OR_HD0 \
	-netdev user,id=umn0,hostfwd=tcp::8023-:23 \
	-device e1000,netdev=umn0
```

UnixWare seems to be finicky with the IDE emulation (in both qemu and VirtualBox), so I opted to use a SCSI adapter for both the CD-ROM and the HDD.

## Installation

You'll need:

- `uw714.CD1.Jun2008.iso` - This is the disc you'll be booting from.
- `uw714.CD2.iso`, also called `Disc02 - Base Operating System Upgrade CD.iso` - This disc contains some additional software that might be useful.
- `uw714mp4.iso` - This disc contains the Maintenance Pack 4, that will upgrade your system.

I also recommend you locate the following:

- `skunkware713.iso`, also called `Disc04 - Linux RPM CD from 7.1.3.iso` - This disc contains a lot of precompiled OSS software, although it's a bit old. 
- `udk714.iso`, also called `Disc05 - UnixWare-OpenServer Development Kit CD.iso` - This disc contains the UnixWare SDK.
- `uw714mp3.iso` - This disc contains the Maintenance Pack 4, that will upgrade your system.

I've found all of the other disc images to be completely optional.

### Procedure

Here's an outline of how I installed the OS on my instance:

1. Install the base OS from `uw714.CD1.Jun2008.iso`.
2. When prompted, insert `uw714.CD2.iso`.
3. Skip the Optional Services CD.
4. Reboot into UnixWare, log into CDE to see if everything's in order.
5. Install the desired SkunkWare packages. (Warning: they're quite old!)
6. Install the UDK from `udk714.iso`. Reboot.
7. Install Maintenance Pack 3 from `uw714mp3.iso` to update the files from UDK. Reboot.
8. Install Maintenance Pack 4 from `uw714mp4.iso`. Reboot.

## Stuff

#### Mounting a cd image

`mount /dev/cdrom/cdrom1 /mnt`

#### Installing a package

> Look under `/mnt/info` for Release Notes!

`install /mnt $PACKAGE_NAME`

#### Shutting down

Use the SysV-style command

`shutdown -i 0 -g 0 -y`

> You must be in the `/` folder for some reason!

## Links

[QEMU machine types and compatibility](https://people.redhat.com/~cohuck/2022/01/05/qemu-machine-types.html)
