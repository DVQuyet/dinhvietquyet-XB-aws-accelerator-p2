# W8-D2 Kubernetes Foundation

## Topics

Today I studied the foundation of containers and Kubernetes.

## Tools installed and verified

- Docker Desktop
- kubectl
- minikube

## Commands practiced

```bash
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\PC> minikube version
minikube version: v1.38.1
commit: c93a4cb9311efc66b90d33ea03f75f2c4120e9b0
PS C:\Users\PC> minikube start --driver=docker
* minikube v1.38.1 on Microsoft Windows 11 Home Single Language 25H2
* Using the docker driver based on user configuration
! Starting v1.39.0, minikube will default to "containerd" container runtime. See #21973 for more info.
* Using Docker Desktop driver with root privileges
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.50 ...
* Downloading Kubernetes v1.35.1 preload ...
    > preloaded-images-k8s-v18-v1...:  272.45 MiB / 272.45 MiB  100.00% 23.37 M
    > gcr.io/k8s-minikube/kicbase...:  519.58 MiB / 519.58 MiB  100.00% 15.77 M
* Creating docker container (CPUs=2, Memory=3072MB) ...
* Preparing Kubernetes v1.35.1 on Docker 29.2.1 ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
PS C:\Users\PC> kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   61s   v1.35.1
PS C:\Users\PC> kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-7d764666f9-swq2s           1/1     Running   0             60s
kube-system   etcd-minikube                      1/1     Running   0             65s
kube-system   kube-apiserver-minikube            1/1     Running   0             65s
kube-system   kube-controller-manager-minikube   1/1     Running   0             65s
kube-system   kube-proxy-vrfst                   1/1     Running   0             61s
kube-system   kube-scheduler-minikube            1/1     Running   0             65s
kube-system   storage-provisioner                1/1     Running   1 (30s ago)   63s
PS C:\Users\PC>