---
title: Kubernetes - Cluster Architecture
search: false
categories:
  - kubernetes
tags:
  - kubernetes
permalink:
last_modified_at: 2021-04-22
---



# {{ page.title }}

> 배를 활용한 이해

## Kubernetes Architecture

> 컨테이너의 형태로 쉽게 배포, 쉽게 다른 앱들과 소통
- 이를 가능하게 하기위해 여러가지 구성이 필요

-
#### Control Plane (Master Node)
> 화물선을 모니터링하고 관리하는 컨트롤선


###### ETCD
- Key-Value Store format database

다른배 어떤컨테이너가 어떤배애, 어떤시간에 세워졌는지,

###### Kube Scheduler
> 크레인으로 컨트롤선에서 컨테이너를 이동

- 적절한 화물선에 적절한 컨테이너를 결정
- 어떤배가 어딨는지, 언제오는지
컨테이너 사이즈들이 다 다른데 화물선마다 선적가능 무게가 다름


스케쥴링 시나리오
컨테이너-10
화물선-4 4 12 16

1. filter nodes
아예 못 채우는 4 4 제거

2. rank nodes
12 > 2, 16 > 4
16에 채우면 12에 채워선 남은 2 보다 더큰 4 가 남기 떄문에 보다 적절


1. non-kubeadm
- /etc/systmemd/system/kube-scheduler.service
2. kubedm
- /etc/kubernetes/manifests/kube-scheduler.yaml
3. from running process
ps -aux | grep kube-scheduler

###### Controller Managers
- Node Controller, Replication Controller, etc.

컨테이너가 부셔졌는지, 부셔졌으면 새로운 컨테이너 전달

1. watch status
2. remidiate situation

node-controller
5초마다 health-check, 40초동안 대답 없으면 kube-apiserver로 전달

replication-Controller
replica들 health-check, 5분 죽었다면 새로운 pod 생성

여러개 controller process 들이 모여서 하나의 controller-manager


1. non-kubeadm
- /etc/systemd/system/kube-controller-manager.service
2. kubeadm
- /etc/kubernetes/manifests/kube-controller-manager.yaml
3. from running process
- ps -aux | grep kube-controller-manager

컨테이너가 부셔졌는지, 부셔졌으면 새로운 컨테이너 전달

###### Kube-API Server
orchestrating all operations within the cluster

pod을 만든다고 가정하면
authenticate Clustervalidate request
retrieve date
update etcd
kube-scheduler가 어떤 화물선에 컨테이너가 올라가야할지 kube-api 한테 알려줌
화물선의 kubelet 이 받고 container 생성


kubeadm > kube-apiserver-master pod

```
/etc/kubernetes/manifests/kube-apiserver.yaml
/etc/systemd/system/kube-apiserver.service
```

#### Worker Node
> 컨테이너를 실을 수 있는 화물선

DNS Service networking solution >> can "container runtime engine" == docker
ContainerD, Rocket

###### Kubelet
> 화물선의 선장

- 컨트롤 선과의 소통
- kube-apiserver로 부터 오는 명령 관리
- 컨테이너 관리

Register node
create pod
monitor node & pod

1. non-kubeadm
> kubeadm doesn't deploy kubelet automatically


###### Kube Proxy

web 컨테이너하나 db 컨테이너 하나 > kube-proxy로 소통


pod network is internal virtual network
pod 의 ip 가 그대로 남을 지는 장담 못함
service를 활용

service는 pod 같이 실체가 있는것이 아님

each node 에 process로 kube-proxy로

IP tables rule 생성

1. kubeadm
- kube-proxy-[random] (deamonset)








#### PODs

Kubernetes는 docker hub 같은 곳에서 Image를 pull 하여 컨테이너 생성

pod = single instance of application

한 Node 안에 여러 Pod 가능
traffic 이 늘어나서 scale을 할 때 Pod를 추가 생성 (Pod 안에 컨테이너를 추가 생성 하는게 아님)


> ###### Multi-Container PODs (side-car pattern)
웹app을 위한 helper-container가 있다면 1개의 Pod 안에 여러 Container가 있을 수 있음
pod는 1개의 pod 안에 있는 container 들에게 shared volume 제공


```
kubectl run nginx --image nginx
kubectl get pods
kubectl get pod
kubectl get po
```

pod만 가지고서는 external user 접근 불가능

_____
-----

## PODs with YAML
> yaml은 indent(2칸, 4칸)를 잘 해야함

**pod-definition-skeleton-akms.yml**
```
apiVersion:
kind:
metadata:

spec:

```

**pod-definition.yml**
```
apiVersion: v1 / v1 / apps/v1 / apps/v1
kind: Pod / Service / ReplicaSet / Deployment
metadata:
  name: myapp-pod
  labels:             (tag 같은 labeling)
    app: myapp
    type: front-end   
spec:                 (dictionary 형태)
  containers:         (list 형태)
    - name: nginx-container
      image: nginx
```

조회
```
kubectl get pods
kubectl describe pod myapp-pod
```


#### DEMO
```
kubectl apply -f pod-definition.yml
kubectl get po -o wide
kubectl delete pod myapp-pod

kubectl run redis --image=redis123 --dry-run=client -o yaml > pod.yaml
kubectl edit pod redis123   (call running app configuration)
```




## ReplicaSets
- HA
- Load Balancing
- Scaling

1. Replication controller
- before

**rc-definition.yml**
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
  ---              (pod-definition.yml 내용이 들어감)
    metadata:
      name:
      labels:
    spec:
      containers:
        - name:
          image:
    ---
  replicas: 3
```

```
kubectl apply -f rc-definition.yml
kubectl get replicationcontroller
kubectl get pods
```

2. ReplicaSet
- after

**replicaset-definition.yml**
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: **front-end**
spec:
  template:
  ---                (pod-definition.yml 내용이 들어감)
    metadata:
      name: myapp-pod
      labels:
    spec:
      containers:
        - name:
          image:
    ---
  replicas: 3

  selector:
    matchLabel:     (수많은 pod 중에서 골라줌)
      type: **front-end**
```

```
kubectl create -f replicaset-definition.yml
kubectl get replicaset
```

```
replicas: 3 > replicas: 6
kubectl replace -f replicaset-deifinition.yml

kubectl scale --replicas=6 -f replicaset-definitionl.yml
kubectl scale --replicas=6 replicaset myapp-replicaset
```



## Deployments
내일 ~















.
