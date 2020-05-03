---
layout: post
title: Install Kubeflow on a single-node Kubernetes v1.18.2 cluster
categories:
  - Tutorial
tags:
  - Kubeflow
  - Kubernetes
  - GPU
  - Ubuntu
excerpt_separator:  <!--more-->
---
## Intro
This installation is based on:
- [kfctl v1.0.2](https://github.com/kubeflow/kfctl/releases/tag/v1.0.2) - [control plane](https://github.com/kubeflow/kfctl) for deploying and managing `Kubeflow`
- [kfctl_k8s_istio config](https://www.kubeflow.org/docs/started/k8s/kfctl-k8s-istio/) - vanilla deployment of `Kubeflow` with all its core components without any external dependencies
- [single-node Kubernetes v1.18.2 cluster](https://illya13.github.io/RL/tutorial/2020/05/01/installing-kubernetes-with-gpu-on-ubuntu-20.html) that was created in previous tutorial

Other goal is to have GPU support. So please check other tutorials:
- [Installing TensorFlow 2 with GPU and Docker support on Ubuntu 20.04 LTS](https://illya13.github.io/RL/tutorial/2020/04/27/installing-tensorflow-in-docker-on-ubuntu-20.html)
- [Installing PyTorch 1.5 with GPU and Docker support on Ubuntu 20.04 LTS](https://illya13.github.io/RL/tutorial/2020/04/28/installing-pytorch-on-ubuntu-20.html)
<!--more-->

## Install kfctl v1.0.2
Assuming Linux/Ubuntu OS
```bash
> wget https://github.com/kubeflow/kfctl/releases/download/v1.0.2/kfctl_v1.0.2-0-ga476281_linux.tar.gz
> tar xzf kfctl_v1.0.2-0-ga476281_linux.tar.gz
```

Optionally add the `kfctl` binary to your path or move it to a folder in `PATH`.
```bash
> mv kfctl .local/bin/
```

## Set env variables and run installation
Change `BASE_DIR` below if required
```bash
> export KF_NAME=kubeflow
> export BASE_DIR=/opt/ubuntu/
> export KF_DIR=${BASE_DIR}/${KF_NAME}
> export CONFIG_URI="https://raw.githubusercontent.com/kubeflow/manifests/v1.0-branch/kfdef/kfctl_k8s_istio.v1.0.2.yaml"
> mkdir -p ${KF_DIR}
> cd ${KF_DIR}
> kfctl apply -V -f ${CONFIG_URI}

INFO[0000] Downloading https://raw.githubusercontent.com/kubeflow/manifests/v1.0-branch/kfdef/kfctl_k8s_istio.v1.0.2.yaml to /tmp/026088338/tmp.yaml  filename="utils/k8utils.go:172"
INFO[0000] Downloading https://raw.githubusercontent.com/kubeflow/manifests/v1.0-branch/kfdef/kfctl_k8s_istio.v1.0.2.yaml to /tmp/959385033/tmp_app.yaml  filename="loaders/loaders.go:71"
...
configmap/seldon-config created
service/seldon-webhook-service created
deployment.apps/seldon-controller-manager created
application.app.k8s.io/seldon-core-operator created
certificate.cert-manager.io/seldon-serving-cert created
issuer.cert-manager.io/seldon-selfsigned-issuer created
validatingwebhookconfiguration.admissionregistration.k8s.io/seldon-validating-webhook-configuration-kubeflow created
INFO[0058] Successfully applied application seldon-core-operator  filename="kustomize/kustomize.go:209"
INFO[0058] Applied the configuration Successfully!       filename="cmd/apply.go:72"
```

Wait deployment and check results
```bash
> kubectl -n kubeflow get all
```

Disable anonymous usage reporting if required
```bash
> kubectl -n kubeflow get all|grep spartakus

pod/spartakus-volunteer-5978bf56f-nlht9                            1/1     Running            0          94m
deployment.apps/spartakus-volunteer                           1/1     1            1           94m
replicaset.apps/spartakus-volunteer-5978bf56f                            1         1         1       94m

> kubectl -n kubeflow delete deployment spartakus-volunteer
``` 

Note. I'm observing deployment issues with `ml-pipeline-viewer-controller`.

## Kubeflow Dashboard