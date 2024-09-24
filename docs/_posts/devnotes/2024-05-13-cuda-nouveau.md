---
title: CUDA Installation - Turning off Nouveau kernel driver
date: 2024-05-13 21:33:27 +0200
tags:
   - cuda
layout: tag

---

## CUDA Installation Problem: 
### WARNING: The Nouveau kernel driver is currently in use by your system.  This driver is incompatible with the NVIDIA driver, and must be disabled before proceeding.

```
(Answer: Continue installation)
-> Performing CC sanity check with CC="/usr/bin/cc".
-> Performing CC check.
WARNING: The Nouveau kernel driver is currently in use by your system.  This driver is incompatible with the NVIDIA driver, and must be disabled before proceeding.
-> Nouveau can usually be disabled by adding files to the modprobe configuration directories and rebuilding the initramfs.

Would you like nvidia-installer to attempt to create these modprobe configuration files for you? (Answer: Yes)
-> One or more modprobe configuration files to disable Nouveau have been written.  You will need to reboot your system and possibly rebuild the initramfs before these changes can take effect.  Note if you later wish to reenable Nouveau, you will need to delete these files: /usr/lib/modprobe.d/nvidia-installer-disable-nouveau.conf, /etc/modprobe.d/nvidia-installer-disable-nouveau.conf
-> nvidia-installer is not able to perform some of the sanity checks which detect potential installation problems while Nouveau is loaded. Would you like to continue installation without these sanity checks, or abort installation, confirm that Nouveau has been properly disabled, and attempt installation again later? (Answer: Abort installation)
-> Initramfs scan complete.
-> The initramfs will likely need to be rebuilt due to the following condition(s):
  * nvidia-installer attempted to disable Nouveau.

Would you like to rebuild the initramfs? (Answer: Rebuild initramfs)
-> /usr/sbin/update-initramfs requires a file path argument, but none was given.
-> Processing the initramfs:
-> Executing: /usr/sbin/update-initramfs -u  
-> done
ERROR: Installation has failed.  Please see the file '/var/log/nvidia-installer.log' for details.  You may find suggestions on fixing installation problems in the README available on the Linux driver download page at www.nvidia.com.

```

## Solution

```
$ sudo vim /etc/default/grub
```
Turn off nouveau by adding this line to the grub file

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nouveau.modeset=0"
```

Don't forget the follow-up command to actually change the configuration:
```
$ sudo update-grub
```