---
title: Kubernetes - JSON Path
description:
search: true
categories:
  - kubernetes
tags:
  - kubernetes
permalink:

last_modified_at:
---


## Application Failure
> draw a map or chart

### Check Accessibility
`curl http://web-service-ip:node-port`
`kubectl get endpoint`

### Check Service Status
`kubectl describe service web-service`

### Check POD
```
kubectl get pod
kubectl describe pod web
kubectl logs web -f --previous 
```

### Check Dependent Service/Apps
```
kubectl get svc
kubectl describe svc
```

## Control Plane Failure

### Check Node Status
```
kubectl get nodes
kubectl get pods
```

### Check Controlplane Pods
```
kubectl get pods -n kube-system
```

### Check Controlplane Services
```
service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status

service kubelet status
service kube-proxy status
```

### Check Service Logs
```
kubectl logs kube-apiserver-master -n kube-system
sudo journalctl -u kube-apiserver
```

## Worker Node Failure

### CHeck Node Status
```
kubectl get nodes
kubectl describe node worker-1
```

### Check Node
```
top
df -h
```

### Check Kubelet Status
```
service kubelet status
sudo jornalctl -u kubelet
```

### Check Certificates
```
openssl x509 -in /var/lib/kubelet/worker-1.cert -text
```

## Networking


## JSONPATH
https://mmumshad.github.io/json-path-quiz/index.html#!/?questions=questionskub1
https://mmumshad.github.io/json-path-quiz/index.html#!/?questions=questionskub2
.
