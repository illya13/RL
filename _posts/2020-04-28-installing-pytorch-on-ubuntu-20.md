---
layout: post
title: Installing PyTorch 1.5 with GPU and Docker support on Ubuntu 20.04 LTS
categories:
  - Tutorial
tags:
  - PyTorch
  - GPU
  - Ubuntu
  - Docker
excerpt_separator:  <!--more-->
---
## Intro
According to [https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/) and [https://github.com/NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker)
the following has to be installed:
- python: `>= 3.6`
- python package managers: `Anaconda` or `pip`
- docker: `> 19.03`

NVIDIA software:
- [NVIDIA GPU drivers](https://www.nvidia.com/drivers)
- [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit): `10.2`
- [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-docker)

Please also note:
- CUDA `10.2` requires GCC `<= 8`

<!--more-->

## Install compatible version of gcc 
Ubuntu 20.04 LTS comes with:
- gcc: `9`

So the first thing we need to do is to install compatible versions of `gcc` i.e. `gcc 8`.

Install `gcc's`. But let's use `gcc 9` for now as it will be used to install GPU Drivers.
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

## Install python and python package managers
There are multiple ways how to manage `python` versions and envs. 
I've selected [pyenv](https://github.com/pyenv/pyenv) + [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv) 
 
```bash
> sudo apt-get install -y zlib1g-dev libbz2-dev libreadline-dev libssl-dev libsqlite3-dev libffi-dev
> pyenv install 3.8.2
> pyenv virtualenv 3.8.2 torch
> pyenv global torch

> python -V
Python 3.8.2
```   

## NVIDIA GPU drivers
Get and run installer.
```bash
> sudo bash NVIDIA-Linux-x86_64-440.82.run

ERROR: The Nouveau kernel driver is currently in use by your system.  This driver is incompatible with the NVIDIA driver, and must be disabled before proceeding.  Please consult the NVIDIA
         driver README and your Linux distribution's documentation for details on how to correctly disable the Nouveau kernel driver.
```

... follow the instructions to blacklist Nouveau kernel driver. Then run:
```bash
> sudo update-initramfs -u
> sudo reboot
````
Retry installation, ignore `Xorg` warnings and configuration if you are working from command line only.
```bash
> sudo bash NVIDIA-Linux-x86_64-440.82.run
```

Note you still need `gcc-9` to install drivers. At the end you should get
`Installation of the NVIDIA Accelerated Graphics Driver for Linux-x86_64 (version: 440.82) is now complete.`

Verify installation:
```bash
> nvidia-smi

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
> sudo bash cuda_10.2.89_440.33.01_linux.run

x CUDA Installer                                                               x
x - [ ] Driver                                                                 x
x      [ ] 440.33.01                                                           x
x + [X] CUDA Toolkit 10.2                                                      x
x   [X] CUDA Samples 10.2                                                      x
x   [X] CUDA Demo Suite 10.2                                                   x
x   [X] CUDA Documentation 10.2                                                x
x   Options                                                                    x
x   Install                                                                    x
```

Please make sure that
```
 -   PATH includes /usr/local/cuda-10.2/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-10.2/lib64, or, add /usr/local/cuda-10.2/lib64 to /etc/ld.so.conf and run ldconfig as root
```

Unzip the cuDNN package
```bash
> tar -xzvf cudnn-10.2-linux-x64-v7.6.5.32.tgz
```

Copy the following files into the CUDA Toolkit directory, and change the file permissions.
```bash
> sudo cp cuda/include/cudnn.h /usr/local/cuda/include
> sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
> sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```

I had to make the following changes:
```bash
> rm /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn.so.7 /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn.so
> sudo ln -s libcudnn.so.7.6.5 /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn.so.7
> sudo ln -s libcudnn.so.7 /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn.so
> sudo ldconfig
```

## PyTorch
```bash
> pip install --upgrade pip
> pip install torch torchvision
```

Check GPU support is enabled, and you can access your GPU:
```bash
> python -c "from __future__ import print_function; import torch; print(torch.cuda.is_available())"

True
```
Also to benchmark your GPU you can do the following:
```bash
> git clone https://github.com/ryujaehun/pytorch-gpu-benchmark
> pip install psutil cufflinks plotly pandas matplotlib
> cd pytorch-gpu-benchmark/
> python benchmark_models.py
```

So you can compare your results with some [well-known numbers](https://github.com/ryujaehun/pytorch-gpu-benchmark). 
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
|    0     84404      C   ...ubuntu/.pyenv/versions/torch/bin/python  7989MiB |
+-----------------------------------------------------------------------------
```

## Docker support on Ubuntu 20.04 LTS
There are number of `docker` packages available
- `docker-ce` package from docker.com
- `docker.io` package provided by Canonical
- `docker` package provided by Red Hat

Docker versions `> 19.03` are [supported](https://github.com/NVIDIA/nvidia-docker/wiki) by NVIDIA Container Toolkit.  

At time of writing the official Ubuntu's `docker.io` is the best option to use. Just run
```bash
> sudo apt install docker-compose
```

Check `docker` version
```bash
> docker version
Client:
 Version:           19.03.8
 API version:       1.40
...

Server:
 Engine:
  Version:          19.03.8
  API version:      1.40 (minimum version 1.12)
...  
``` 

## NVIDIA Container Toolkit
Add the package repositories
```bash
> distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
> curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
> curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

At this point I had to clean up `/etc/apt/sources.list.d/` folder. 
```bash
> cd /etc/apt/sources.list.d/; rm docker.list*
```

Continue installation
```bash
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

## Validating installation
Run `nvidia-smi` in a docker
```bash
> docker run --gpus all --rm nvidia/cuda nvidia-smi

+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.82       Driver Version: 440.82       CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 107...  Off  | 00000000:09:00.0 Off |                  N/A |
|  0%   50C    P0    35W / 180W |      0MiB /  8118MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

Run `PyTorch` in a docker
```bash
> docker run -it --rm --gpus all pytorch/pytorch python -c "from __future__ import print_function; import torch; print(torch.cuda.is_available())"

True
```

Run benchmark in a docker
```bash
> docker run --gpus all --shm-size=512M -it --rm -v /home/ubuntu/pytorch-gpu-benchmark:/pytorch-gpu-benchmark pytorch/pytorch
(in a docker) > pip install psutil cufflinks plotly pandas matplotlib
(in a docker) > python /pytorch-gpu-benchmark/benchmark_models.py
```
Fix the path `/home/ubuntu/pytorch-gpu-benchmark` above to your local folder.