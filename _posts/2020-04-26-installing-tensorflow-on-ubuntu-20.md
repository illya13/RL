---
layout: post
title: Installing TensorFlow 2 with GPU support on Ubuntu 20.04 LTS
categories:
  - Tutorial
tags:
  - TensorFlow
  - TF
  - GPU
  - Ubuntu
excerpt_separator:  <!--more-->
---
# Intro
According to [https://www.tensorflow.org/install](https://www.tensorflow.org/install)
and [https://www.tensorflow.org/install/gpu](https://www.tensorflow.org/install/gpu)
the following has to be installed:
- python: `3.5 – 3.7`
- pip: `> 19.0`

NVIDIA software:
- NVIDIA GPU drivers: `> 418.x`
- CUDA Toolkit: `10.1`
- cuDNN SDK: `>= 7.6`

Please also note:
- CUDA `10.1` and `10.2` requires GCC `<= 8`

At time of writing there are following versions available:
- TensorFlow: `2.1.x` <fix me>
- [NVIDIA GPU drivers](https://www.nvidia.com/drivers): `440.82`
- [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit): `10.2.89` (with GPU drivers `440.33`) and `10.1.243` (with GPU drivers `418.87`)
- [cuDNN SDK](https://developer.nvidia.com/cudnn): `v7.6.5`

<!--more-->
# Compatible versions of python and gcc 
Ubuntu 20.04 LTS comes with:
- python: `3.8`
- gcc: `9`

So the first thing we need to do is to install compatible versions of `python` and `gcc`

```bash
> sudo apt -y install build-essential
> sudo apt -y install gcc-8 g++-8 gcc-9 g++-9

> sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 8 --slave /usr/bin/g++ g++ /usr/bin/g++-8
> sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 9 --slave /usr/bin/g++ g++ /usr/bin/g++-9

> sudo update-alternatives --config gcc

There are 2 choices for the alternative gcc (providing /usr/bin/gcc).

  Selection    Path            Priority   Status
------------------------------------------------------------
* 0            /usr/bin/gcc-9   9         auto mode
  1            /usr/bin/gcc-8   8         manual mode
  2            /usr/bin/gcc-9   9         manual mode

Press <enter> to keep the current choice[*], or type selection number: 0
update-alternatives: using /usr/bin/gcc-9 to provide /usr/bin/gcc (gcc) in auto mode
```

```bash
> sudo add-apt-repository ppa:deadsnakes/ppa
Press [ENTER]
```

```bash
> sudo apt-get update
> sudo apt-get install python3.7

> sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.7 7
> sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.8 8

> sudo update-alternatives --config python
There are 2 choices for the alternative python (providing /usr/bin/python).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3.8   8         auto mode
  1            /usr/bin/python3.7   7         manual mode
  2            /usr/bin/python3.8   8         manual mode

Press <enter> to keep the current choice[*], or type selection number: 1
update-alternatives: using /usr/bin/python3.7 to provide /usr/bin/python (python) in manual mode

> python --version
Python 3.7.7
```   
# NVIDIA GPU drivers
```bash
> sudo bash NVIDIA-Linux-x86_64-440.82.run

ERROR: The Nouveau kernel driver is currently in use by your system.  This driver is incompatible with the NVIDIA driver, and must be disabled before proceeding.  Please consult the NVIDIA
         driver README and your Linux distribution's documentation for details on how to correctly disable the Nouveau kernel driver.
```

... follow the instructions to blacklist Nouveau kernel driver ...

```bash
> sudo update-initramfs -u
> sudo reboot
````
Retry installation, ignore `Xorg` warnings and configuration if you are working only from command line.
Note you still need `gcc-9` to install drivers. At the end you should get
`Installation of the NVIDIA Accelerated Graphics Driver for Linux-x86_64 (version: 440.82) is now complete.`

```bash
> sudo bash NVIDIA-Linux-x86_64-440.82.run
```

Verify installation:
```bash
> nvidia-smi

Sun Apr 26 07:38:29 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.82       Driver Version: 440.82       CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 107...  Off  | 00000000:09:00.0 Off |                  N/A |
|  0%   59C    P0    38W / 180W |      0MiB /  8118MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
``` 

```bash
> sudo update-alternatives --config gcc

There are 2 choices for the alternative gcc (providing /usr/bin/gcc).

  Selection    Path            Priority   Status
------------------------------------------------------------
* 0            /usr/bin/gcc-9   9         auto mode
  1            /usr/bin/gcc-8   8         manual mode
  2            /usr/bin/gcc-9   9         manual mode

Press <enter> to keep the current choice[*], or type selection number: 1
update-alternatives: using /usr/bin/gcc-8 to provide /usr/bin/gcc (gcc) in manual mode

gcc --version
gcc (Ubuntu 8.4.0-3ubuntu2) 8.4.0
```

