---
title: CUDA Installation - shuting down the X server
date: 2024-05-13 11:23:01 +0700
tags:
   - cuda
layout: tag

---

## CUDA Installation failed.
### You appear to be running an X server.  Installing the NVIDIA driver while X is running is not recommended
```
$ sudo sh cuda_12.4.1_550.54.15_linux.run
 Installation failed. See log at /var/log/cuda-installer.log for details.
```

Check the `cuda-installer.log`.
```
$ less /var/log/cuda-installer.log
[INFO]: Driver not installed.
[INFO]: Checking compiler version...
[INFO]: gcc location: /usr/bin/gcc
[INFO]: gcc version: gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04) 
[INFO]: Initializing menu
[INFO]: nvidia-fs.setKOVersion(2.19.7)
[INFO]: Setup complete
[INFO]: Installing: Driver
[INFO]: Installing: 550.54.15
[INFO]: Executing NVIDIA-Linux-x86_64-550.54.15.run --ui=none --no-questions --accept-license --disable-nouveau --no-cc-version-check --install-libglvnd  2>&1
[INFO]: Finished with code: 256
[ERROR]: Install of driver component failed. Consult the driver log at /var/log/nvidia-installer.log for more details.
[ERROR]: Install of 550.54.15 failed, quitting

```


Check the `nvidia-installer.log`, for the message stating: You appear to be running an X server.  **Installing the NVIDIA driver while X is running is not recommended**.
```
$ less /var/log/nvidia-installer.log
nvidia-installer log file '/var/log/nvidia-installer.log'
creation time: Wed May  8 19:45:52 2024
installer version: 550.54.15

PATH: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

nvidia-installer command line:
    ./nvidia-installer
    --ui=none
    --no-questions
    --accept-license
    --disable-nouveau
    --no-cc-version-check
    --install-libglvnd

Using built-in stream user interface
-> Detected 32 CPUs online; setting concurrency level to 32.
-> Scanning the initramfs with lsinitramfs...
-> Executing: /usr/bin/lsinitramfs   -l /boot/initrd.img-6.5.0-28-generic
-> The file '/tmp/.X0-lock' exists and appears to contain the process ID '2059' of a running X server.
-> You appear to be running an X server.  Installing the NVIDIA driver while X is running is not recommended, as doing so may prevent the installer from detecting some potential installation problems, and it may not be possible to start new graphics applications after a new driver is installed.  If you choose to continue installation, it is highly recommended that you reboot your computer after installation to use the newly installed driver. (Answer: Abort installation)
ERROR: Installation has failed.  Please see the file '/var/log/nvidia-installer.log' for details.  You may find suggestions on fixing installation problems in the README available on the Linux driver download page at www.nvidia.com.
/var/log/nvidia-installer.log
```

## Shut down the X server
The common x server on ubuntu can be gdm, gdm3 or lightdm.

Check the X server status
```
$ sudo service gdm status
● gdm.service - GNOME Display Manager
     Loaded: loaded (/lib/systemd/system/gdm.service; static)
     Active: active (running) since Wed 2024-05-08 14:56:43 CEST; 4h 57min ago
    Process: 1419 ExecStartPre=/usr/share/gdm/generate-config (code=exited, status=0/SUCCESS)
   Main PID: 1424 (gdm3)
      Tasks: 3 (limit: 76936)
     Memory: 6.2M
        CPU: 99ms
     CGroup: /system.slice/gdm.service
             └─1424 /usr/sbin/gdm3

maj 08 14:56:43 belay systemd[1]: Starting GNOME Display Manager...
maj 08 14:56:43 belay systemd[1]: Started GNOME Display Manager.
maj 08 14:56:43 belay gdm-launch-environment][1429]: pam_unix(gdm-launch-environment:session): session opened for user gdm(uid=128) by (uid=0)
maj 08 14:57:11 belay gdm-password][1784]: gkr-pam: unable to locate daemon control file
maj 08 14:57:11 belay gdm-password][1784]: gkr-pam: stashed password to try later in open session
maj 08 14:57:11 belay gdm-password][1784]: pam_unix(gdm-password:session): session opened for user belay(uid=1000) by (uid=0)
```

Shut down the X server, and you screen will turn black immediatedly.
```
$ sudo service gdm stop
```

### Go to non-GUI interface
Press the keyboard: `Alt` + `F2`, and switch to the terminal.
Run the CUDA again

```
sudo sh cuda_12.4.1_550.54.15_linux.run
```
