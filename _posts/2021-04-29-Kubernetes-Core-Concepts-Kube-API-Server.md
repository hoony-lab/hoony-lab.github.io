---
title: Kubernetes - Kube API Server
search: true
categories:
  - kubernetes
tags:
  - kubernetes
permalink: /pages
last_modified:
---



# {{ title }}

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
Install

Download
```
https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz
```
Extract
tar xzvf

Run
./etcd

- cluster 의 정보들을 저장
  - node, pod, config, secret, account role, role-binding, etc.

- etcd server에 올라가면



> kubernetes 설치는 2가지 방법
1. 바닥부터 설치
2. kuadmtool을 활용한 설치

etcd.service
> --adverties-client == http://{ip}:port

pod 확인 > etcd-master
--initial-cluster 옵션에 다른 컨트롤러 노드 ip 확인


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

###### Kube-API Server
orchestrating all operations within the cluster

pod을 만든다고 가정하면
authenticate Clustervalidate request
retrieve date
update etcd
kube-scheduler가 어떤 화물선에 컨테이너가 올라가야할지 kube-api 한테 알려줌
화물선의 kubelet 이 받고 container 생성
