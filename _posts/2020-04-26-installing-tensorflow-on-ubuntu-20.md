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
## Intro
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
- CUDA `10.1` requires GCC `<= 8`

At time of writing there are following versions available:
- TensorFlow: `2.1.0`
- [NVIDIA GPU drivers](https://www.nvidia.com/drivers): `440.82`
- [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit): `10.1.243` (with GPU drivers `418.87`)
- [cuDNN SDK](https://developer.nvidia.com/cudnn): `v7.6.5`

<!--more-->
## Compatible versions of python and gcc 
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

There are multiple ways how to manage python version and envs. 
I've selected [pyenv](https://github.com/pyenv/pyenv) + [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv) 
 
```bash
> sudo apt-get install -y zlib1g-dev libbz2-dev libreadline-dev libssl-dev libsqlite3-dev libffi-dev
> pyenv install 3.7.7
> pyenv virtualenv 3.7.7 tf2
> pyenv global tf2

> python -V
Python 3.7.7
```   
## NVIDIA GPU drivers
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
## CUDA Toolkit and cuDNN
Switch `gcc` to `gcc-8`
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

Run installer, accept license, deselect Driver 
```bash
> sudo bash cuda_10.1.243_418.87.00_linux.run

x CUDA Installer                                                               x
x - [ ] Driver                                                                 x
x      [ ] 418.87.00                                                           x
x + [X] CUDA Toolkit 10.1                                                      x
x   [X] CUDA Samples 10.1                                                      x
x   [X] CUDA Demo Suite 10.1                                                   x
x   [X] CUDA Documentation 10.1                                                x
x   Options                                                                    x
x   Install                                                                    x
```

Please make sure that
```
 -   PATH includes /usr/local/cuda-10.1/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-10.1/lib64, or, add /usr/local/cuda-10.1/lib64 to /etc/ld.so.conf and run ldconfig as root
```

Unzip the cuDNN package
```bash
> tar -xzvf cudnn-x.x-linux-x64-v8.x.x.x.tgz
```

Copy the following files into the CUDA Toolkit directory, and change the file permissions.
```bash
> sudo cp cuda/include/cudnn.h /usr/local/cuda/include
> sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
> sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

I had to make the following changes
```bash
> rm /usr/local/cuda-10.1/targets/x86_64-linux/lib/libcudnn.so.7 /usr/local/cuda-10.1/targets/x86_64-linux/lib/libcudnn.so
> sudo ln -s libcudnn.so.7.6.5 /usr/local/cuda-10.1/targets/x86_64-linux/lib/libcudnn.so.7
> sudo ln -s libcudnn.so.7 /usr/local/cuda-10.1/targets/x86_64-linux/lib/libcudnn.so
> sudo ldconfig
```
## TensorFlow 2 with GPU support
```bash
> pip install --upgrade pip
> pip install tensorflow
> pip install tensorflow-gpu
```

Check GPU support is enabled, and you can access your GPU
```bash
> python -c "import tensorflow as tf; tf.config.list_physical_devices('GPU')"
```
Ignore the warning `successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero`
Also to a draft benchmark your GPU you still (repo is not maintained) do the following:
```bash
> git clone https://github.com/tensorflow/benchmarks
> python benchmarks/scripts/tf_cnn_benchmarks/tf_cnn_benchmarks.py --num_gpus=1 --model resnet50 --batch_size 64
```

So you can compare your results with some [well-known numbers](https://www.leadergpu.com/tensorflow_resnet50_benchmark). 
You also can run `nvidia-smi` in parallel to check GPU load.
```bash
> watch nvidia-smi

+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.82       Driver Version: 440.82       CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 107...  Off  | 00000000:09:00.0 Off |                  N/A |
| 39%   63C    P2   179W / 180W |   7999MiB /  8118MiB |     99%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0     84404      C   ...e/ubuntu/.pyenv/versions/tf2/bin/python  7989MiB |
+-----------------------------------------------------------------------------
```

