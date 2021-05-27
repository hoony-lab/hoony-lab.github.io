---
title: Kubernetes - ETCD
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
