# Create a Kubernetes Cluster Using Kubeadm on Ubuntu 22.04
In this guide, we will set up a Kubernetes cluster from scratch using Ansible and Kubeadm, and then deploy a containerized Nginx application to it.

## Introduction
Kubernetes is a container orchestration system that manages containers at scale. Initially developed by Google based on its experience running containers in production, Kubernetes is open source and actively developed by a community around the world.

Kubeadm automates the installation and configuration of Kubernetes components such as the API server, Controller Manager, and Kube DNS. It does not, however, create users or handle the installation of operating-system-level dependencies and their configuration. For these preliminary tasks, it is possible to use a configuration management tool like Ansible or SaltStack. Using these tools makes creating additional clusters or recreating existing clusters much simpler and less error-prone.
## Goals
Our cluster will include the following physical/Virtual resources:
- **One Control Plane node**

    The control plane node (a node in Kubernetes refers to a server) is responsible for managing the state of the cluster. 


- **Two or more worker nodes**

    Worker nodes are the servers where our workloads (i.e., containerized applications and services) will run. A worker will continue to run your workload once they’re assigned to it, even if the control plane goes down once scheduling is complete. We can increase the cluster’s capacity by adding workers.

## Prerequisites
- An SSH key pair on our local Linux/macOS/BSD machine. 
- Three servers running Ubuntu 22.04 with at least 2GB RAM and 2 vCPUs each. We should be able to SSH into each server as the root user with your SSH key pair.
- Ansible should be installed on our local machine. 

## Installation Steps
**Step 1** — Update the Ansible Inventory File

**Step 2** — Create a non-Root User on All Remote Servers (in our case, we already have a non-root user, so skipping this step).
```bash
ansible-playbook -i hosts create_user.yaml
```

**Step 3** — Install Kubernetes Dependencies on All Nodes in the Cluster (Containerd is target Runtime for this deployment).
```bash
ansible-playbook -i hosts containerd/install_dependencies_all_nodes.yaml
```

**Step 4** — Initialize the K8S cluster from the Master Node (Control Plane) 
```bash
ansible-playbook -i hosts containerd/init_k8s_cluster.yaml
ansible-playbook -i hosts containerd/config_k8s_cluster.yaml
ansible-playbook -i hosts containerd/init_cni_k8s_cluster.yaml
```

**Step 5** — Add Worker Nodes to the Cluster
```bash
ansible-playbook -i hosts provision_worker_nodes.yaml
```

**Step 6** — Add Labels to the Cluster Nodes
```bash
kubectl get nodes --show-labels
kubectl label nodes worker1 disktype=ssd ostype=normal appstype=user
kubectl label nodes worker2 disktype=ssd ostype=realtime appstype=user
kubectl label nodes worker3 disktype=ssd ostype=normal appstype=observability
```

**Step 7** — Verify the Cluster
```bash
ssh aida@control_plane_ip
```
Then execute the following command to get the status of the cluster:
```bash
kubectl get nodes -o wide
```

**Step 8a** — Running an application on the cluster with runC
```bash
kubectl create deployment nginx --image=nginx
```

**Step 8b** — Running an application on the cluster with Kata
```bash
kubectl apply -f containerd/kata-runtime-class.yaml
kubectl apply -f containerd/example-kata-pod.yaml
```