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
Docker versions `> 19.03` are supported by NVIDIA Container Toolkit  

<!--more-->
At time of writing the official Ubuntu's `docker.io` is the best option to use. Just run
```bash
> sudo apt install docker-compose

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
