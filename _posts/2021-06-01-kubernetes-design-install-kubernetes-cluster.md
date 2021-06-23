---
title: Kubernetes - Design and Install a Kubernetes Cluster
description:
search: true
categories:
  - kubernetes
tags:
  - kubernetes
permalink:

last_modified_at: 2021-06-02
---


## Design a Kubernetes cluster

- Purpose
  - Education
    - Minikube
    - Single node cluster w/ kubeadm/GCP/AWS
  - Dev & Test
    - Multi-node cluster w/ Single Master and Mulitple workers
    - kubeadm/GCP/AWS/AKS
  - Hosting Production Apps
    - consider HA
    - upto 5000 nodes
    - upto 150,000 pods in cluster
    - upto 300,000 total containers
    - upto 100 pods per node
- Cloud or OnPrem
  - Kubeadm for OnPrem
  - GKE for GCP
  - KOPS for AWS
  - AKS for Azure
- Workloads
  - Storages
  - Nodes
    - Virtual or Physical Machines
    - minimum of 4 node cluster
    - Master vs Worker nodes
    - Linux X86_64 Arch.
  - How many ?
  - What kind ?


## Choose Kuberenetes Infrastrucure Config

Linux
- binary (O)
- tools (O)

Windows : Hyper-V, Virtualbox, ...
- binary (X)
- tools (O)

Minikube
- single node

Kubeadm
- multi node


Turnkey Solutions : easy to deploy and manage a k8s cluster PRIVATELY
- OpenShift
- Cloud Foundry Container Runtime
  - BOSH
- VMware Cloud PKS
- Vagrant

Hosted Solutions
- GKE (Google Container Engine)
- AKS (Azure Kubernetes Service)
- EKS (Amazon Elastic Continer Service for Kubernetes)
- OpenShift Online


## Choose a Network Solution


## HA Kubernetes Cluster

> HA for avoid "single point of failure"

### API server : A-A
split traffic between API server

### Scheduler : A-S (election)

### ETCD

Leader Election - RAFT

Quorum = N/2 + 1

recommended to have odd number of nodes

### Install from the scratch
https://www.youtube.com/watch?v=uUupRagM7m0&list=PL2We04F3Y_41jYdadX55fdJplDvgNGENo

## Install with Kubeadm

> https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

1. Control/Worker nodes (Provision VM)
2. install docker on Control/Worker nodes
3. install Kubeadm on Control node
4. initilize Control node
5. POD network in Control/Worker nodes
6. join Worker node to Control node


## Provision Infrastructure


## Secure Cluster Communication


## Kuberenetes Release Binaries


## Install Kubernetes Master Nodes


## Install Kubernetes Worker Nodes


## TLS Bootstrapping a Node


## Node end-to-end tests

End to End tests is **no longer part of the CKA exam**

> https://www.youtube.com/watch?v=-ovJrIIED88&list=PL2We04F3Y_41jYdadX55fdJplDvgNGENo&index=18

## Run & Analyze end-to-end test

End to End tests is **no longer part of the CKA exam**

> https://www.youtube.com/watch?v=-ovJrIIED88&list=PL2We04F3Y_41jYdadX55fdJplDvgNGENo&index=18






.
