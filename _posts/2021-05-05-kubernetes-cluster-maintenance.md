---
title: Kubernetes - Cluster Maintenance
description:
search: false
categories:
  - kubernetes
tags:
  - kubernetes
permalink:
date: 2021-05-05
last_modified_at: 2021-05-05
---


## Cluster upgrade process

### OS Upgrades

```
kubectl drain <node-name>     (gracefully terminated from node then move to other node)
kubectl cordon <node-name>    (cordon = Mark node as unschedulable, 저지선, 비상경계선)
kubectl uncordon <node-name>
```

### Kubernetes Software Versions

v1.11.3
<major>.<minor>.<patch>

July 2015 v1.0


## Operating System upgrades

cannot be higher then kube-apiserver version
controller-manager, kube-scheduler
kubelet, kube-proxy

버젼은 세개씩 supported
버젼업은 한단계씩 하는걸 recommended


1. upgrade control node
  - cannot modify, new pod deploy,
2. upgrade worker node
  - at the same time : cause downtime
  - one by one : rolling update
  - add new nodes : convenient if your on cloud env.

```
// control
kubeadm upgrade plan    (show current, latest, kubeadm version)

apt-get upgrade -y kubeadm=1.12.0-00
kubectl upgrade apply <version>
kubectl get nodes       (show kubelet version)

apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet

// worker
kubectl drain <node-name>
kubectl uncordon <node-name>
apt-get upgrade -y kubeadm=1.12.0-00
apt-get upgrade -y kubelet=1.12.0-00
kubeadm upgrade <node-name> config --kubelet-version v1.12.0
systemctl restart kubelet
```


## Backup and restore methodologies


.
