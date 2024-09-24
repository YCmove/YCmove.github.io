---
title: Completely Remove/Uninstall all CUDA driver
date: 2024-06-01 13:00:29 +0100
tags:
   - cuda
layout: tag

---

<!-- # Remove/Uninstall all CUDA driver -->

### Remove nvidia driver
```
sudo apt-get --purge remove "*cuda*" "*cublas*" "*cufft*" "*cufile*" "*curand*" \
 "*cusolver*" "*cusparse*" "*gds-tools*" "*npp*" "*nvjpeg*" "nsight*" "*nvvm*"
sudo apt-get --purge remove "*nvidia*" "libxnvctrl*"
sudo apt --purge remove cudnn*
```


### Remove PPA

```
rm /etc/apt/sources.list.d/cuda* /etc/apt/sources.list.d/nvidia-container*
sudo apt update
sudo apt upgrade
```


### Cleanup the leftover
```
sudo apt autoremove
sudo apt autoclean
```
- `autoremove` is used to remove packages that were automatically installed to satisfy dependencies for other packages and are now no longer needed as dependencies changed or the package(s) needing them were removed in the meantime.
- `autoclean` remove the .deb file of uninstalled packages or old versions, usually located in `/var/cache/apt/`.


### Check if they are all removed
After reboot your PC `sudo reboot`, there should be no cuda related packages
```
dpkg -l | grep cuda
sudo find /usr -iname "*cuda*" -type d 2> /dev/null
```


### Reference
- [removing cuda toolkit and driver](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#removing-cuda-toolkit-and-driver)
- [autoclean and autoremove](https://www.quora.com/Linux-whats-the-difference-between-autoclean-and-autoremove)