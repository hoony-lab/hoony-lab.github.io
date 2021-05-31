---
title: Kubernetes - Core Concepts
description:
search: true
categories:
  - kubernetes
tags:
  - kubernetes
permalink:

last_modified_at: 2021-04-29
---

# {{ page.title }}

> 배를 활용한 이해

## Kubernetes Architecture
> 컨테이너의 형태로 쉽게 배포, 쉽게 다른 앱들과 소통
- 이를 가능하게 하기위해 여러가지 구성이 필요

### Control Plane (Master Node)
> 화물선을 모니터링하고 관리하는 컨트롤선


#### ETCD
- Key-Value Store format database

다른배 어떤컨테이너가 어떤배애, 어떤시간에 세워졌는지,

#### Kube Scheduler
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

#### Controller Managers
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

#### Kube-API Server
orchestrating all operations within the cluster

pod을 만든다고 가정하면
authenticate Clustervalidate request
retrieve data
update etcd
kube-scheduler가 어떤 화물선에 컨테이너가 올라가야할지 kube-api 한테 알려줌
화물선의 kubelet 이 받고 container 생성


kubeadm > kube-apiserver-master pod

```
/etc/kubernetes/manifests/kube-apiserver.yaml
/etc/systemd/system/kube-apiserver.service
```

### Worker Node
> 컨테이너를 실을 수 있는 화물선

DNS Service networking solution >> can "container runtime engine" == docker
ContainerD, Rocket

#### Kubelet
> 화물선의 선장

- 컨트롤 선과의 소통
- kube-apiserver로 부터 오는 명령 관리
- 컨테이너 관리

Register node
create pod
monitor node & pod

1. non-kubeadm
> kubeadm doesn't deploy kubelet automatically


#### Kube Proxy

web 컨테이너하나 db 컨테이너 하나 > kube-proxy로 소통


pod network is internal virtual network
pod 의 ip 가 그대로 남을 지는 장담 못함
service를 활용

service는 pod 같이 실체가 있는것이 아님

each node 에 process로 kube-proxy로

IP tables rule 생성

1. kubeadm
- kube-proxy-[random] (deamonset)








## PODs

Kubernetes는 docker hub 같은 곳에서 Image를 pull 하여 컨테이너 생성

pod = single instance of application

한 Node 안에 여러 Pod 가능
traffic 이 늘어나서 scale을 할 때 Pod를 추가 생성 (Pod 안에 컨테이너를 추가 생성 하는게 아님)


>  Multi-Container PODs (side-car pattern)
> - 웹app을 위한 helper-container가 있다면 1개의 Pod 안에 여러 Container가 있을 수 있음
> - pod는 1개의 pod 안에 있는 container 들에게 shared volume 제공


```
kubectl run nginx --image nginx
kubectl get pods
kubectl get pod
kubectl get po
```

**pod만 가지고서는 external user 접근 불가능**


## PODs with YAML
> yaml은 indent(2칸, 4칸)를 잘 해야함

**pod-definition-skeleton-akms.yml**
```yaml
apiVersion:
kind:
metadata:

spec:

```

**pod-definition.yml**
```yaml
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


### DEMO
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
```yaml
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
```yaml
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

deploy app in prod env

- Deployment
  - ReplicaSet
    - Pod


**deployment-definition.yml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
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




## Namespaces

> 집 안에 pod, service, deployment 가 있는 격


1. default
2. kube-system
3. kube-public

production이 커지면 namespace도 구분을 잘해야ㅐ한다
4. dev
- web-pod
- db-service
5. prod
- web-pod
- db-service

`db-service.dev.svc.cluster.local`
`<service-name>.<namespace>.<service>.<domain>.<domain>`

각각의 policy
- quota 설정


```
kubectl get pods --namespace=kube-system

kubectl create -f pod-deifinition.yaml
kubectl create -f pod-deifinition.yaml --namespace=dev
```


**pod-definition-ns.yml**
```yaml
apiVersion: v1 / v1 / apps/v1 / apps/v1
kind: Pod / Service / ReplicaSet / Deployment
metadata:
  name: myapp-pod
  ** namespace: dev **
  labels:             (tag 같은 labeling)
    app: myapp
    type: front-end   
spec:                 (dictionary 형태)
  containers:         (list 형태)
    - name: nginx-container
      image: nginx
```


**namespace-definition.yml**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```
kubectl create -f namespace-deifinition.yml
```


```
kubectl create namespace dev
```




- 기본 namespace 설정

kubectl config set-context $(kubectl config current-context) --namespace=dev



Resource Quota

**compute-quota-definition.yml**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

```
kubectl create -f compute-quota-definition.yml
```




## Services
service enable communication between other components
- service enable commnunitation to front-end
- service enable commnunitation to back-end


external communication
- web app in pod.

labtop - 192.180.1.10
node - 192.168.1.2
pod - 10.244.0.0
service - port 30008

```
curl http://192/169.1.2:30008
```

### Service Types

#### NodePort
> TargetPort on pod > Port on Service > NodePort on node

- TargetPort : pod 10.244.0.2:80
- Port : 10.106.1.12:80
- NodePort : 30000 ~ 32767

**pod-definition.yml**
```yaml
labels:
  ** app: myapp **
```

**service-nodeport-definition.yml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    ** app: myapp **
    type: front-end
```

```
kubectl create -f service-nodeport-definition.yaml
kubectl get services
curl htpp://192.168.0.1:30008
```

random algorithm  
session affinity : yes

> 3개의 node가 있을때  
curl http://192.168.0.2:30008  
curl http://192.168.0.3:30008  
curl http://192.168.0.4:30008  
모두 같은 service를 통하여 라벨링된 앱으로 traffic 전달


#### ClusterIP

> FE > BE > redis

pod의 ip는 static 하지 않음
single interface를 만들어 각 pod group에 접근해야함


**pod-definition.yml**
```yaml
labels:
  ** app: myapp **
```


```yaml
# service-clusterip-definition.yml

apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
    type: back-end
```


#### LoadBalancer

> aws, azure, gcp (어떤 클라우드는 지원 안할 수도 있음)

voting-app in 3 pods in 1 deployment (http://example-vote.com)
  - node 1
  - node 2

result-app 3 pods in 1 deployment (http://example-result.com)
  - node 3
  - node 4  




```yaml
# service-loadbalancer-definition.yml

apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: back-end
```



> ## Imperative vs Declarative
> 1. Imperative (명령)
>  - selects each steps
>    - provision VM  
>    - install nginxedit config port 8080  
>    - edit config web path  
>  ```
>  kubectl run --image=nginx nginx
>  kubectl create deployment --image=nginx nginx
>  kubectl expose pod nginx --port 80
>  kubectl expose deployment nginx --port 80
>  kubectl scale deployment nginx --replicas=5
>  kubectl set image deployment nginx nginx=nginx:1.18
>  kubectl create/replace/delete -f nginx.yaml
>  kubectl run nginx --image=nginx --dry-run=client -o yaml > foo.yaml
>  ```
>  ```
>  kubectl edit deployment nginx
>  kubectl replace -f nginx.yaml
>  kubectl replace --force -f nginx.yaml
>  ```
>
>  2. Declarative (manifest.yaml)
>    VM name: web  
>    Database: nginx  
>    Port: 8080  
>    Path: /var/www/nginx  
>    Code: GIT Repo - X  
>
>  ```
>  kubectl apply -f nginx.yaml     (nginx.yaml.isempty() ? create : update)
>  ```
