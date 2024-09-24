---
title: Install multiple CUDA on one machine
date: 2024-06-02 10:44:20 +0300
tags:
   - cuda
layout: tag

---
<!-- # Install multiple CUDA on one machine -->

### Check your GPU is is CUDA-capable
```
$ lspci | grep VGA
VGA compatible controller: NVIDIA Corporation GA102 [GeForce RTX 3090] (rev a1)
```


### Check your OS
```
$ uname -m && cat /etc/*release
x86_64
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.4 LTS"
```
Where `x86_64` means that it is a 64-bit version of the x86 instruction.


### Check gcc compiler
```
$ gcc --version
gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
Copyright (C) 2021 Free Software Foundation, Inc.
```
You usually don't have to worry about the gcc version; the default logic comes with your OS and should function without a problem. Ubuntu 22.04 uses gcc-11 by default.


### Ensuring Proper Kernel Header and Development Package are Installed
```
$ uname -r
6.5.0-28-generic
```
You can check your OS distribution and corresponding kernel on [system-requirements](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/#system-requirements).


### Switch the kernel for Ubuntu 22.04
Following the Nvidia, for `Ubuntu 22.04.z (z <= 3) LTS`, the support kernel should be `6.2.0-26` and the gcc version should be `11.4.0`.

To switch the kernel version, follow the instructions provided in this article: [how to change kernel](https://gist.github.com/YCmove/1b8fc14503d56845ed98b4b7e1f25a11)


### Download the first CUDA installer
First and the foremost, prioritize the installation of the **latest** or **higher** version of the CUDA driver, as it typically maintains [backward compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/index.html).

```
wget https://developer.download.nvidia.com/compute/cuda/12.4.1/local_installers/cuda_12.4.1_550.54.15_linux.run
sudo sh cuda_12.4.1_550.54.15_linux.run
```
- You would probably run into **X server issue**, check out [this article](https://gist.github.com/YCmove/60abe36a4b7a4162eec6d97f034828cd) to solve it.
- You would probably run into **nouveau issue**, check out [this article](https://gist.github.com/YCmove/3723f799ba1e75a39541cdf0ce1e8d0c) to solve it.

Do not choose nvidia-fs if your GPU is not supported.
```
┌──────────────────────────────────────────────────────────────────────────────┐
│ CUDA Installer                                                               │
│ - [X] Driver                                                                 │
│      [X] 550.54.15                                                           │
│ + [X] CUDA Toolkit 12.4                                                      │
│   [X] CUDA Demo Suite 12.4                                                   │
│   [X] CUDA Documentation 12.4                                                │
│ - [] Kernel Objects                                                         │
│      [] nvidia-fs                                                           │
│   Options                                                                    │
│   Install                                                                    │
│                                                                              │
│ Up/Down: Move | Left/Right: Expand | 'Enter': Select | 'A': Advanced options │
└──────────────────────────────────────────────────────────────────────────────┘

```


## Set Enviroment Variables for CUDA
Add three lines in `/home/{username}/.bashrc`
```
export CUDA_HOME=/usr/local/cuda-12.4
export PATH=/usr/local/cuda-12.4/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64:$LD_LIBRARY_PATH
```


## Install another CUDA 11.8
```
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run
sudo sh cuda_11.8.0_520.61.05_linux.run
```

Install the CUDA toolkit only
```
┌──────────────────────────────────────────────────────────────────────────────┐
│ CUDA Installer                                                               │
│ - [] Driver                                                                 │
│      [] 5xx.xx.xx                                                           │
│ + [X] CUDA Toolkit 11.8                                                      │
│   [] CUDA Demo Suite 11.8                                                   │
│   [] CUDA Documentation 11.8                                                │
│ - [] Kernel Objects                                                         │
│      [] nvidia-fs                                                           │
│   Options                                                                    │
│   Install                                                                    │
│                                                                              │
│ Up/Down: Move | Left/Right: Expand | 'Enter': Select | 'A': Advanced options │
```

## Install the environment-modules
We use environment modules to simply the shell initiallization for dual CUDA toolkits.
```
sudo apt-get update
sudo apt-get install environment-modules
echo -e "source /usr/share/modules/init/profile.sh" >> ~/.bashrc 
source ~/.bashrc
```
It is not recommand to overwrite your system's `/etc/profile.d/modules.sh` by apt-installed third-party package [configuration](https://modules.readthedocs.io/en/stable/INSTALL.html#configuration)

## Initialize the dual CUDA modules
Run this [init_two_cuda_modules.sh](https://github.com/YCmove/toolbox/blob/main/init_two_cuda_modules.sh) as follows:
```
bash init_two_cuda_modules.sh {cuda_default_version} {cuda_another_version}
```

For example, if you want cuda-12.4 as default and cuda-11.8 as an alternative:
```
bash init_two_cuda_modules.sh 12.4 11.8
```
This script will automatically generate the required module configurations for you.

## Switch between two CUDA versions
Unload the current module first before you load the other module.
```
module unload cuda/12.4
module load cuda/11.8
```



## References
- [Environment Modules Doc](https://modules.readthedocs.io/en/stable/module.html)
- [Steps to manage multiple CUDA environments](https://gist.github.com/garg-aayush/156ec6ddda3d62e2c0ddad00b7e66956)
- [Multiple Version of CUDA Libraries On The Same Machine](https://blog.kovalevskyi.com/multiple-version-of-cuda-libraries-on-the-same-machine-b9502d50ae77)