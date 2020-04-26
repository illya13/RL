---
layout: post
title: Installing TensorFlow 2 with GPU support on Ubuntu 20.04 LTS in Docker
categories:
  - Tutorial
tags:
  - TensorFlow
  - TF
  - GPU
  - Ubuntu
  - Docker
excerpt_separator:  <!--more-->
---
## Intro
Please check [Installing TensorFlow 2 with GPU support on Ubuntu 20.04 LTS](https://illya13.github.io/RL/tutorial/2020/04/26/installing-tensorflow-on-ubuntu-20.html) for the intro.

According to [https://www.tensorflow.org/install/docker](https://www.tensorflow.org/install/docker)
and [https://github.com/NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker)
the following has to be installed:
- docker: `> 19.03`
- [NVIDIA GPU drivers](https://www.nvidia.com/drivers)
- [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-docker)

## Docker in Ubuntu 20.04 LTS
There are number of `docker` packages available
- `docker-ce` package from docker.com
- `docker.io` package provided by Canonical
- `docker` package provided by Red Hat
Docker versions `> 19.03` are [supported](https://github.com/NVIDIA/nvidia-docker/wiki) by NVIDIA Container Toolkit.  

<!--more-->
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

Run `TensorFlow` in a docker
```bash
> docker run -it --rm --gpus all tensorflow/tensorflow:latest-gpu python -c "import tensorflow as tf; tf.config.list_physical_devices('GPU')"
```

Run benchmark in a docker
```bash
> docker run --gpus all -it --rm -v /home/ubuntu/benchmarks:/benchmarks tensorflow/tensorflow:latest-gpu python benchmarks/scripts/tf_cnn_benchmarks/tf_cnn_benchmarks.py --num_gpus=1 --model resnet50 --batch_size 64
```