---
title: "Installing an OS to a physical hard disk using a VM"
date:
categories:
 - hardware
tags:
 - vm
 - disk
 - external
---

Alright, this is a quickie. In an ill-fated attempt to install OS/2 Warp 4.52 to the [bespoke IGEL D210 thin client](https://jack23247.github.io/blog/sysadm/alpine-igel/), I ~~re~~learned that you can use a physical hard disk (or to a CF card, or any other block device for that matter) with a virtual machine. And it's not even that hard!

With QEMU you can simply do this (as `root`):

```
# qemu-system-i386 \
    -m 256 \
    -boot d \
    -cdrom mcp2-refresh-boot-en.iso \
    -drive format=raw,file=/dev/sdb \
    -net nic,model=rtl8139
```

The `format=raw` part is necessary to inform qemu that we want it to treat the file as a raw block device rather than as a disk image.

With VirtualBox you can do the following (as `root`):

```
# VBoxManage internalcommands createrawvmdk \
    -filename igel-cf.vmdk \
    -rawdisk /dev/sdb
```

This will create a fake VMDK that points to the raw device (`/dev/sdb` in this case), provided you run VirtualBox with superuser privileges.

<!-- I'm afraid it's time for me to go, kids: an undergraduate student in Genoa is trying to install Windows 95 onto `/dev/null`! CAPTAIN CRAPWARE -->