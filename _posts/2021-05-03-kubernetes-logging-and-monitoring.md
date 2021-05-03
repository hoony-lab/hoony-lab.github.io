---
title: Kubernetes - Logging and Monitoring
description:
search: false
categories:
  - kubernetes
tags:
  - kubernetes
permalink:

last_modified_at: 2021-05-03
---

> node level metrics : CPU, memory, network, disk utilization  
pod level metrics : CPU, memory consumption

> ~~Heapster is deprecated~~

metric server (in-memory)  
- kubelet은 instructions 을 apiserver에서 받음  

  - cAdvisor 포함, pod metrics를 metrics server 로 전달


## Monitor Cluster Components

```
kubectl top node
```

## Monitor Applications

```
kubectl top pod
```

## Monitor Cluster Components Logs


## Application Logs

```
kubectl logs -f <pod-name>
kubectl logs -f <pod-name> <container-name>
```

.
