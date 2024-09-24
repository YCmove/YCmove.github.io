---
title: Automatic mounting a disk during boot time
date: 2024-05-29 23:12:11 +0200
tags:
   - unix
layout: tag

---

## Filesystem Status Check
Check the current filesystem status and how they are mounted on your computer.
```
$ mount
/dev/nvme0n1p2 on / type ext4 (rw,relatime,errors=remount-ro)
/dev/nvme0n1p1 on /boot/efi type vfat (rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
```
We can read this as **the device `/dev/nvme0n1p2` is mounted on a mount point `/`**

## Check the connected device

```
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0     4K  1 loop /snap/bare/5
loop1         7:1    0 311,3M  1 loop /snap/code/158
loop2         7:2    0 311,3M  1 loop /snap/code/159
sda           8:0    0  16,4T  0 disk 
└─sda1        8:1    0  16,4T  0 part 
nvme0n1     259:0    0   1,8T  0 disk 
├─nvme0n1p1 259:1    0   487M  0 part /boot/efi
└─nvme0n1p2 259:2    0   1,8T  0 part /
```
Multiple "loopX" partitions is because of "snaps," which is the Canonical's universal package management system. But for now, you can just ignore them.
Here we want to auto-mount `sda1` and when booting.





## UUID (Universally Unique Identifier)
list your devices by UUID use blkid

```
$ sudo blkid
/dev/nvme0n1p2: UUID="ba3fb267-681f-4414-a479-886d13be256f" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="9daa9a7c-ed95-4d9b-92ce-2f336072b18a"
/dev/nvme0n1p1: UUID="D523-44B9" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="3288d544-c593-4777-a3aa-18c1a93030f8"
/dev/sda1: UUID="ac876a5c-83ac-4029-bad1-3ae5fa7c3831" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="3d070b69-4ea7-491c-aa84-21d1e73a26fb"
```

## Create Mounting Points
```
sudo mkdir /media/mystorage
```


## Filesystem Table /etc/fstab
`/etc/fstab` holds the essential details for automating partition mounting. 
- `file system`: UUID of the disk
- `mount point`: Mount Point
- `type`: Type of file system
- `options`: Mount options of access to the device/partition (see the man page for mount).
- `dump`: Enable or disable backing up of the device/partition (the command dump). This field is usually set to 0, which disables it.
- `pass`: Controls the order in which fsck checks the device/partition for errors at boot time. The root device should be 1. Other partitions should be 2, or 0 to disable checking.
```
$ /etc/fstab
# <file system>                           <mount point>  <type>   <options>        <dump>  <pass>
UUID=xxxxxxxx-xxxx-xxxx-xxxx-886d13be256f /               ext4    errors=remount-ro  0      1
UUID=xxxx-xxxx                            /boot/efi       vfat    umask=0077         0      1
```
Add new line for device `/dev/sda1`
```
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /media/mystorage ext4 errors=remount-ro 0 2
```

## Mount all filesystems mentioned in fstab
```
mount -a
```