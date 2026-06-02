# W8-D2 Kubernetes Foundation

- Kubernetes is a container orchestration platform used to deploy, run, scale, update, and manage containerized applications.

- In Kubernetes, applications are not managed directly as single containers. Instead, Kubernetes manages objects such as Pods, Deployments, Services, ConfigMaps, Secrets, and NetworkPolicies.

- A Pod is the smallest deployable unit in Kubernetes. A Pod usually contains one application container, but it can also contain multiple containers that need to work together.

- Pods are temporary resources. They can be deleted, recreated, or moved to another node. Because of this, applications should not depend directly on a specific Pod IP address.

- A Deployment is commonly used to manage Pods. It helps keep the desired number of Pods running and can recreate Pods automatically when they fail.

- A Service provides a stable network endpoint for accessing a group of Pods. Since Pod IPs can change, clients should connect to a Service instead of connecting directly to Pods.

- Service uses selectors to find the correct Pods. For example, a Service can target all Pods with a specific label such as `app: backend`.

- Common Service types include:
  - `ClusterIP`: exposes the Service only inside the cluster.
  - `NodePort`: exposes the Service on a static port of each node.
  - `LoadBalancer`: exposes the Service externally through a cloud load balancer.
  - `ExternalName`: maps the Service to an external DNS name.

- A ConfigMap is used to store non-sensitive configuration data as key-value pairs.

- ConfigMaps help separate application configuration from container images. For example, values such as `APP_ENV`, `LOG_LEVEL`, or `DATABASE_HOST` can be stored in a ConfigMap.

- Pods can use ConfigMaps as environment variables or as configuration files mounted into the container.

- A Secret is used to store sensitive data such as passwords, tokens, private keys, and TLS certificates.

- Secrets are similar to ConfigMaps, but they are intended for confidential data. Sensitive values should not be hard-coded in application code or container images.

- Secrets can also be used by Pods as environment variables or mounted files. However, Secrets are not fully secure by default, so production environments should use RBAC and encryption at rest.

- A NetworkPolicy is used to control network traffic between Pods and between Pods and external systems.

- NetworkPolicy works like a firewall rule for Kubernetes workloads. It can define which Pods are allowed to communicate with each other and through which ports.

- Ingress means traffic entering a Pod.

- Egress means traffic leaving a Pod.

- By default, if no NetworkPolicy exists, Pods can usually communicate freely inside the namespace. When NetworkPolicies are applied, only explicitly allowed traffic is permitted.

- NetworkPolicy requires a network plugin that supports policy enforcement. If the cluster network plugin does not support NetworkPolicy, creating a NetworkPolicy resource will not have any effect.

- Today I also verified the local Kubernetes environment using Docker Desktop, kubectl, and minikube.

- The local Kubernetes cluster was started successfully with minikube using the Docker driver.

- `kubectl get nodes` showed that the minikube node was in `Ready` status.

- `kubectl get pods -A` showed that the Kubernetes system Pods were running successfully.

- Main takeaway: Kubernetes runs applications inside Pods, exposes them through Services, manages configuration with ConfigMaps and Secrets, and controls communication using NetworkPolicies.

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