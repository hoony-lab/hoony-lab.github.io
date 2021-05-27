---
title: Kubernetes - Core Concepts 2
description:
search: true
categories:
  - kubernetes
tags:
  - kubernetes
permalink:

last_modified_at: 2021-05-03
---


## Scheduling

### How scheduling works

**pod-definition.yaml**
```yaml
apiVersion:
kind:
metadata:
  name: nginx
  labels:
    name: nginx-name
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 8080
      nodeName:                   (일반적으로 생략되고, 자동작성해줌)
```

```
kubectl get pod -o wide
```



### If No scheduler
- manually schedule pods

```yaml
pod-bind-definition.yaml

apiVersion:
kind: Binding
metadata:
  name: nginx
  labels:
    name: nginx-name
target:
  apiVersion: v1
  kind: Node
  ** nodeName: node02 **
```


## Labels and Selectors

```yaml
replicaset-definition.yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-1

spec:
  replicas: 2
  selector:
    matchLabels:
      name: nginx-name
      class: mammal  
      kind: cat  
      color: green

    template:
      metadata:
        labels:
          name: nginx-name
          class: Mammal  
          kind: cat  
          color: green
    spec:
      containers:
        - name: nginx-container
          image: nginx
          ports:
            - containerPort: 8080
```

```
kubectl get po --show-labels
kubectl get po -l color=green
kubectl get po -l color=green,kind=cat
kubectl get po -l color=green --no-headers | wc -l
```


## Taints and Tolerations (오염과 관용)
> 어떤 모기는 모기약뿌린 사람한테 안가지만, 어떤 슈퍼모기는 모기약뿌린 사람한테도 감
- 사람은 node, 모기는 pod, 슈퍼모기는 tolerant 된 pod

```
kubectl taint nodes [node-name] [key=value:taint-effect]
- taint-effect = [NoSchedule | PreferNoSchedule | NoExecute]
  - NoExecute: node에
  - NoSchedule:
  - PreferNoSchedule:
```

```
$ kubectl taint nodes node1 app=blue:NoSchedule
```

```yaml
pod-definition.yml

spec:
  containers:
    - name:
      image:
    tolerations:
      key: app
      operator: Equal
      value: blue
      effect: NoSchedule
```

controlplane node에 taint가 설정되어서 pod들이 controlplane node 에 가지 않음


## Node Selectors
> set limitation on pods

node 1: size 10
node 2: size 5
node 3: size 5
위와 같은 구성일 때

```
kubectl label nodes <node-name> <label-key>=<label-value>
```

```yaml
pod-nodeselector-definition.yml

...
spec:
  containers:
    - name:
      image:
  nodeSelector:
    ????????????
```


# kubectl explain pod --recursive

## Node Affinity (친밀감, 관련성)

> pod host on particuler node

```yaml
pod-nodeselector-definition.yml

...
spec:
  containers:
    - name:
      image:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:     (라벨이 없으면 안함)
      preferredDuringSchedulingIgnoredDuringExecution:    (라벨이 없어도 가능)
      preferredDuringSchedulingRequiredDuringExecution:    (라벨이 없으면 앱 삭제)
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In / NotIn / Exists / ...
            values:
            - Large
```






## Resoruce Requirements and Limits

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```






## DeamonSets

> one pod copy of ALL each node

kube-apiserver에 의해 생성됨

- use case
  - monitoring agent
  - kube-proxy
  - weave-net

```yaml
daemon-set-definition.yaml (similar to replicaset)

apiVersion: apps/v1
kind: DeamonSet
metadata:
  name: monitoring-agent
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```

```
kubectl create -f daemon-set-definition.yaml
```

after v1.12



## Static Pods

> /etc/kubernetes/manifests/<pod-definition-name>.yaml

자동으로 읽고 pod 생성/수정/삭제

1. service
  - --pod-manifest-path=/etc/kubernetes/manifest


2. kubelet.service
  - --config=<kubeconfig-name.yaml>   >   staticPodPath: /etc/kubernetes/manifest

static pod 는 각각의 kubelet에 의해 생성됨

kube-apiserver 에서는 read-only

kubeadm 은 control-plane에 static pod 형태로 구성

> ps -ef | grep kubelet     >     find "--config"


## Multiple Scheduler


### Custom Scheduler

```yaml
pod-scheduler-definition.yaml

...
spec:
  containers:
    - name:
      image:
  schedulerName: my-custom-scheduler
```

```
kubectl get events
```












.
