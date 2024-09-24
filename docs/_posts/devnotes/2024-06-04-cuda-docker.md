---
title: Docker could not select device driver with capabilities
date: 2024-06-04 23:19:38 +0600
tags:
   - cuda
layout: tag

---

## Error Message
```
docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]].
```

## First, create a docker sudoer group and add your account
```
sudo groupadd docker
sudo usermod -aG docker $USER
```

## Second, enable [rootless](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#rootless-mode) mode
```
nvidia-ctk runtime configure --runtime=docker --config=$HOME/.config/docker/daemon.json
systemctl --user restart docker
sudo nvidia-ctk config --set nvidia-container-cli.no-cgroups --in-place
```

## Test
```
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```
```
Mon Sep 16 13:13:49 2024       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 560.35.03              Driver Version: 560.35.03      CUDA Version: 12.6     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3090        Off |   00000000:04:00.0 Off |                  N/A |
|  0%   28C    P8             12W /  350W |      15MiB /  24576MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA GeForce RTX 3080        Off |   00000000:07:00.0  On |                  N/A |
|  0%   47C    P8             24W /  320W |     410MiB /  10240MiB |     13%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```