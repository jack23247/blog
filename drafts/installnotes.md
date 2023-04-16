# Installing SCO UnixWare 7.1.4 in qemu



## Set-up

```
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

- uw714.CD1.Jun2008.iso
- uw714.CD2.iso (Disc02 - Base Operating System Upgrade CD.iso)
- uw714mp4.iso 

Recommended:

- skunkware713.iso (Disc04 - Linux RPM CD from 7.1.3.iso)
- udk714.iso (Disc05 - UnixWare-OpenServer Development Kit CD.iso)
- uw714mp3.iso

Optional:

All of the other CD images.

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
