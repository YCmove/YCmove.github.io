---
title: How to Safely Switch Between Kernel Versions
date: 2024-05-10 19:38:01 +0700
tags:
   - unix
layout: tag

# hidden: true
---

## Check the current kernel
```
$ dpkg --list 'linux-image-*'
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                  Version             Architecture Description
+++-=====================================-===================-============-===============================>
ii  linux-image-6.5.0-18-generic          6.5.0-18.18~22.04.1 amd64        Signed kernel image generic
ii  linux-image-6.5.0-28-generic          6.5.0-28.29~22.04.1 amd64        Signed kernel image generic
ii  linux-image-generic-hwe-22.04         6.5.0.28.29~22.04.1 amd64        Generic Linux kernel image
un  linux-image-unsigned-6.5.0-18-generic <none>              <none>       (no description available)
un  linux-image-unsigned-6.5.0-28-generic <none>              <none>       (no description available)
```


## Check the apt kernel packages

```
$ apt list --all-versions linux-image* | less
linux-image-5.15.0-100-generic/jammy-updates,jammy-security 5.15.0-100.110 amd64
linux-image-5.15.0-101-generic/jammy-updates,jammy-security 5.15.0-101.111 amd64
...
linux-image-6.2.0-25-generic/jammy-updates,jammy-security 6.2.0-25.25~22.04.2 amd64
```


## Install the specific kernel
```
sudo apt update
sudo apt install linux-image-6.2.0-26-generic
```

## Optional

Only install the headers package if you plan to compile kernel functions by yourself.
```
sudo apt install linux-headers-6.2.0-26-generic
```

If you need extra drivers
```
sudo apt install linux-modules-extra-6.2.0-26-generic
```

## Switching Kernel at Booting
```
sudo vim /etc/default/grub
GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.2.0-26-generic"
sudo update-grub
sudo reboot
```