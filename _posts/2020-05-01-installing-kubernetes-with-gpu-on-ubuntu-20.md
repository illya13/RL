---
layout: post
title: Installing Kubernetes with GPU support on Ubuntu 20.04 LTS
categories:
  - Tutorial
tags:
  - Kubernetes
  - GPU
  - Ubuntu
excerpt_separator:  <!--more-->
---
## Intro
There are [multiple ways](https://ubuntu.com/kubernetes/install) to install Kubernetes on Ubuntu 20.04.
The [easiest](https://ubuntu.com/kubernetes/install#single-node) but not the best way is to use [MicroK8s](https://microk8s.io/) for single node setup. 
At this point I would not recommend using `MicroK8s` if you need GPU support:
- I personally failed to make it working with GPU support
    - [standard K8s GPU example doesn't work](https://github.com/ubuntu/microk8s/issues/448)
    - [Revert to containerd 1.2.5 until we get gpu to work on 1.3.0](https://github.com/ubuntu/microk8s/pull/813)
- it is trying to do too many things on multiple platforms
- snap-based installs are still questionable.

So we'll go with `Kubeadm` install in this tutorial.

## Installation Prerequisites
Before continuing Kubernetes installation process, please make sure you already have installed:
- [NVIDIA GPU drivers](https://www.nvidia.com/drivers)
- docker: `> 19.03`
- [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-docker)
<!--more-->

Check docker NVIDIA support 
```bash
> docker info | grep nvidia

 Runtimes: runc nvidia
 Default Runtime: nvidia
```

If not please check my other tutorials and install missing bits:
- [Installing TensorFlow 2 with GPU support on Ubuntu 20.04 LTS in Docker](https://illya13.github.io/RL/tutorial/2020/04/27/installing-tensorflow-on-ubuntu-20-in-docker.html)
- [Installing PyTorch 1.5 with GPU and Docker support on Ubuntu 20.04 LTS](https://illya13.github.io/RL/tutorial/2020/04/28/installing-pytorch-on-ubuntu-20.html)

## Installation Options
Our installation will include the following:
- [Kubernetes](https://kubernetes.io/): `v1.18+`
- [CoreDNS](https://github.com/coredns/deployment) - enabled by default 
- [Cilium](https://docs.cilium.io/en/stable/) - as Pod network
- [NVIDIA device plugin](https://github.com/NVIDIA/k8s-device-plugin) - run GPU enabled containers

Optional: 
- [Helm](https://github.com/helm/helm)
- [Kubernetes Dashboard](https://github.com/kubernetes/dashboard) (TBD)
- Local Docker Registry (TBD)

## Install kubeadm
Our installation procedure will be based on:
- [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Installing kubeadm, kubelet and kubectl:
```bash
> sudo apt-get update && sudo apt-get install -y apt-transport-https curl
> curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
> cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
> sudo apt-get update
> sudo apt-get install -y kubelet kubeadm kubectl
> sudo apt-mark hold kubelet kubeadm kubectl
``` 

Modify `/etc/docker/daemon.json` and add the following
```bash
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
```

My full `/etc/docker/daemon.json`
```json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia": {
      "path": "/usr/bin/nvidia-container-runtime",
      "runtimeArgs": []
    }
  },
  "insecure-registries": ["localhost:32000"]
}
```

Restart docker
```bash
> sudo systemctl daemon-reload
> sudo service docker restart
``` 

## Using kubeadm to Create a Cluster
- [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

Init cluster, copy config to a current user, launch cilium Pod network, untainted master
```bash
> sudo kubeadm init --pod-network-cidr=10.217.0.0/16
> mkdir -p $HOME/.kube
> sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> sudo chown $(id -u):$(id -g) $HOME/.kube/config
> kubectl create -f https://raw.githubusercontent.com/cilium/cilium/v1.6/install/kubernetes/quick-install.yaml
> kubectl taint nodes --all node-role.kubernetes.io/master-
```

Check status so far
```bash
> kubectl get pods --all-namespaces
> docker ps
```

## NVIDIA device plugin
Install NVIDIA device plugin for Kubernetes:
```bash
> kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/1.0.0-beta6/nvidia-device-plugin.yml
```

Check status so far
```bash
> kubectl get pods --all-namespaces

NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   cilium-operator-6dd6ddbd78-m8xh4       1/1     Running   2          10h
kube-system   cilium-sgxst                           1/1     Running   2          10h
kube-system   coredns-66bff467f8-rcxrb               1/1     Running   1          10h
kube-system   coredns-66bff467f8-tzxkm               1/1     Running   1          10h
kube-system   etcd-rl                                1/1     Running   1          10h
kube-system   kube-apiserver-rl                      1/1     Running   1          10h
kube-system   kube-controller-manager-rl             1/1     Running   1          10h
kube-system   kube-proxy-4dv4l                       1/1     Running   1          10h
kube-system   kube-scheduler-rl                      1/1     Running   1          10h
kube-system   nvidia-device-plugin-daemonset-plbsl   1/1     Running   1          10h
```

```bash
> docker ps

ea36d0c10cc3    k8s_nvidia-device-plugin-ctr_nvidia-device-plugin-daemonset-plbsl_kube-system_7a1ffb88-5c23-44fa-ade3-3518c4f8e14d_1
468ecaea6b7d    k8s_coredns_coredns-66bff467f8-rcxrb_kube-system_db5c7834-212b-44c3-a499-6cd27237b7cf_1
ff607a27a2fe    k8s_coredns_coredns-66bff467f8-tzxkm_kube-system_cdedd3a9-3005-4438-b3fd-a05cafa25b97_1
a911e43a16da    k8s_POD_nvidia-device-plugin-daemonset-plbsl_kube-system_7a1ffb88-5c23-44fa-ade3-3518c4f8e14d_1
393f367d1deb    k8s_POD_coredns-66bff467f8-rcxrb_kube-system_db5c7834-212b-44c3-a499-6cd27237b7cf_1
7e0f07b0021f    k8s_POD_coredns-66bff467f8-tzxkm_kube-system_cdedd3a9-3005-4438-b3fd-a05cafa25b97_1
47ad7c694d5f    k8s_cilium-agent_cilium-sgxst_kube-system_1732281a-7e68-4a99-b98b-9eca6b769e9f_2
91da9f31ee3e    k8s_cilium-operator_cilium-operator-6dd6ddbd78-m8xh4_kube-system_c562d8fc-b23d-4d75-8a7d-f4b8464d497d_2
4476f718c175    k8s_POD_cilium-operator-6dd6ddbd78-m8xh4_kube-system_c562d8fc-b23d-4d75-8a7d-f4b8464d497d_1
8f15fd741356    k8s_kube-proxy_kube-proxy-4dv4l_kube-system_f0074b84-2137-43ca-a223-3706a2b4fcd4_1
ca61b93e2abc    k8s_POD_cilium-sgxst_kube-system_1732281a-7e68-4a99-b98b-9eca6b769e9f_1
9e1b6e973c28    k8s_POD_kube-proxy-4dv4l_kube-system_f0074b84-2137-43ca-a223-3706a2b4fcd4_1
352db42ba835    k8s_kube-apiserver_kube-apiserver-rl_kube-system_eb25054123f2196031edcebd5b3b9c63_1
cf1097a52235    k8s_kube-controller-manager_kube-controller-manager-rl_kube-system_b0adb2ac3ca75e9b6a5a23a700d2df3f_1
bfdfd7e67708    k8s_kube-scheduler_kube-scheduler-rl_kube-system_155707e0c19147c8dc5e997f089c0ad1_1
15d34f015c9b    k8s_etcd_etcd-rl_kube-system_60c09396edb87f94d77681a0385b8731_1
2f9195742b25    k8s_POD_etcd-rl_kube-system_60c09396edb87f94d77681a0385b8731_1
eb39b126fbb8    k8s_POD_kube-scheduler-rl_kube-system_155707e0c19147c8dc5e997f089c0ad1_1
20d4eeaf0966    k8s_POD_kube-controller-manager-rl_kube-system_b0adb2ac3ca75e9b6a5a23a700d2df3f_1
c6ae250bf615    k8s_POD_kube-apiserver-rl_kube-system_eb25054123f2196031edcebd5b3b9c63_1
```

```bash
> kubectl describe nodes

...
Capacity:
...
  nvidia.com/gpu:     1
...
Allocatable:
...
  nvidia.com/gpu:     1
...
Allocated resources:
...
  nvidia.com/gpu     0           0
...
```

## Run GPU-enabled Pod
Create file `nvidia-smi.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nvidia-smi
spec:
  containers:
    - name: nvidia-smi
      image: nvidia/cuda
      command: ["nvidia-smi"]
      resources:
        limits:
          nvidia.com/gpu: 1
```

Create Pod
```bash
> kubectl create -f nvidia-smi.yml

pod/nvidia-smi created
```

Check Pod logs
```bash
> kubectl logs nvidia-smi

+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.82       Driver Version: 440.82       CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 107...  Off  | 00000000:09:00.0 Off |                  N/A |
|  0%   52C    P8    14W / 180W |      0MiB /  8118MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

Delete Pod
```bash
> kubectl delete -f nvidia-smi.yml

pod "nvidia-smi" deleted
```
## Optional Installation
### Install Helm 3
```bash
> curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

Downloading https://get.helm.sh/helm-v3.2.0-linux-amd64.tar.gz
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm

> helm version
version.BuildInfo{Version:"v3.2.0", GitCommit:"e11b7ce3b12db2941e90399e874513fbd24bcb71", GitTreeState:"clean", GoVersion:"go1.13.10"}
```

### Install Kubernetes Web UI (Dashboard)
Our installation procedure will be based on:
- [Deploying the Dashboard UI](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#deploying-the-dashboard-ui)
- [Getting Started](https://github.com/kubernetes/dashboard#getting-started)
- [Access Control](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/README.md)
- [Creating sample user](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md) 

```bash
> kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```
