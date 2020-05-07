---
layout: post
title: Reinstall GPU drivers after Linux kernel upgrade
categories:
  - Tutorial
tags:
  - kernel
  - drivers
  - GPU
  - Ubuntu
excerpt_separator:  <!--more-->
---
## Intro
Sometimes you are getting notifications like this... 
```bash
14 updates can be installed immediately.
11 of these updates are security updates.
To see these additional updates run: apt list --upgradable
```
<!--more-->
... and you do upgrade. 
```bash
> sudo apt-get update
> sudo apt-get upgrade

Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
The following packages have been kept back:
  linux-generic linux-headers-generic linux-image-generic
The following packages will be upgraded:
  landscape-common libldap-2.4-2 libldap-common linux-libc-dev python3-update-manager update-manager-core
6 upgraded, 0 newly installed, 0 to remove and 3 not upgraded.
```

And it usually doesn't affect your environment especially GPU support. But if you'd like to upgrade the kernel and you do `dist-upgrade`
```bash
> sudo apt-get dist-upgrade

Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
The following NEW packages will be installed:
  linux-headers-5.4.0-29 linux-headers-5.4.0-29-generic linux-image-5.4.0-29-generic linux-modules-5.4.0-29-generic linux-modules-extra-5.4.0-29-generic
The following packages will be upgraded:
  linux-generic linux-headers-generic linux-image-generic
3 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
```

and your kernel got upgraded then after reboot you might lose GPU support 
```bash
> nvidia-smi

NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.
```

So you need to reinstall your GPU drivers.

## Reinstall GPU drivers
So just complete GPU driver install steps that are described in details in other tutorials
- [Installing TensorFlow 2 with GPU and Docker support on Ubuntu 20.04 LTS](https://illya13.github.io/RL/tutorial/2020/04/27/installing-tensorflow-in-docker-on-ubuntu-20.html)
- [Installing PyTorch 1.5 with GPU and Docker support on Ubuntu 20.04 LTS](https://illya13.github.io/RL/tutorial/2020/04/28/installing-pytorch-on-ubuntu-20.html)

## Check GPU support
```bash
> nvidia-smi

+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.82       Driver Version: 440.82       CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 107...  Off  | 00000000:09:00.0 Off |                  N/A |
| 35%   53C    P8    14W / 180W |      0MiB /  8118MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

## Remove old kernel (optional)
```bash
> sudo apt autoremove

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be REMOVED:
  linux-headers-5.4.0-26 linux-headers-5.4.0-26-generic linux-image-5.4.0-26-generic linux-modules-5.4.0-26-generic linux-modules-extra-5.4.0-26-generic
0 upgraded, 0 newly installed, 5 to remove and 0 not upgraded.
After this operation, 359 MB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 151549 files and directories currently installed.)
Removing linux-headers-5.4.0-26-generic (5.4.0-26.30) ...
Removing linux-headers-5.4.0-26 (5.4.0-26.30) ...
Removing linux-modules-extra-5.4.0-26-generic (5.4.0-26.30) ...
Removing linux-image-5.4.0-26-generic (5.4.0-26.30) ...
/etc/kernel/postrm.d/initramfs-tools:
update-initramfs: Deleting /boot/initrd.img-5.4.0-26-generic
/etc/kernel/postrm.d/zz-update-grub:
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.4.0-29-generic
Found initrd image: /boot/initrd.img-5.4.0-29-generic
Found linux image: /boot/vmlinuz-5.4.0-28-generic
Found initrd image: /boot/initrd.img-5.4.0-28-generic
done
Removing linux-modules-5.4.0-26-generic (5.4.0-26.30) ...
```
