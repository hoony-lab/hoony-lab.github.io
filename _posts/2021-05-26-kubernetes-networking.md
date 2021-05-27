---
title: Kubernetes - Networking
description:
search: false
categories:
  - kubernetes
tags:
  - kubernetes
permalink:

last_modified_at: 2021-05-26
---


## Prerequisites


### Swicting and Routing

#### Switching

> like 192.168.1.10/24

```
ip link

ip addr add 192.168.1.10/24 dev eth0
ping 192.168.1.10
```

#### Routing

> connect several networks

#### Default Gateway

```
route

ip route add 192.168.2.0/24 via 192.168.1.1

ip route add 192.168.1.0/24 via 192.168.2.1
ip route add default via 192.168.2.1
ip route add 0.0.0.0 via 192.168.2.1
```

### DNS

#### Name Resolution

```bash
# /etc/hosts

192.168.1.1 web
192.168.1.2 db
```

```bash
# /etc/resolv.conf

nameserver 8.8.8.8

nameserver 192.168.1.100
search mycompany.com  prod.mycompany.com
```

#### nslookup command

```
nslookup www.google.com
```

#### dig command

```
dig www.google.com
```

### CoreDNS

> DNS Server solution

> https://coredns.io/manual/toc/


### Tools


### Network Namespaces

process namespace


```
ip netns add red
ip netns add blue

ip link

ip netns exec red ip link
ip -n red link
```

```
arp
route
```

```
ip line add veth-red type veth peer name veth-blue
ip link set veth-red netns red
ip line set veth-blue netns blue
ip -n red addr add 192.168.15.1 dev veth-red
ip -n blue addr add 192.168.15.2 dev veth-blue
ip -n red line set veth-red up
ip -n blue line set veth-blue up

ip -n red link del veth-red
```

#### Linux Bridge

```
ip link add veth-red type veth peer name veth-red-br
ip link add veth-blue type veth peer name veth-blue-br

ip link set veth-red netns red
ip link set veth-red-br master v-net-0

ip link set veth-blue netns blue
ip link set veth-blue-br master v-net-0

ip -n red addr ...


ip addr add 192.168.15.5/24 dev v-net-0
...

```

### Docker Networking

#### none
```
docker run --network none nginx7
```

#### host


#### bridge

> so deep

```
docker network ls
```

### CNI

1. Create Network Namespace
2. Create Bridge Network/Interface
3. Create vEth Pairs (Pipe, Vitual Cable)
4. Attach vEth to Namespace
5. Attach Other vEth to Bridge
6. Assign IP Addresses
7. Bring the interfaces up
8. Enable NAT -IP Masquerade

CRI implement CNI
but Docker doesn't implement CNI, uses CNM(Container Network Model)

## Networking Configuration on Cluster nodes

### IP & FQDN

### Ports

kube-api 6443
kubelet 10250
kube-scheduler 10251
kube-controller-manager 10252
services 30000-32767

```
ip link
ip addr
ip addr add 192.168.1.10/24 dev eth0
ip route
ip route add 192.168.1.0/24 via 192.168.2.1
route
cat /proc/sys/net/ipv4/ip_forward
arp
netstat -pInt
```

## Service networking

?


## POD Networking Concepts

### Networking Model

- Every POD should have an **IP Address**
- Every POD should able to communicate with every other **POD in the same node**
- Every POD should able to communicate with every other **POD on other nodes** without NAT


## CNI in Kubernetes

IPAM = IP Address Management

### CNI - weaveworks

Weave and Weave peers can be deployed as Deamonset on each nodes

```
kubectl apply -f \
  "https://cloud.weave.works/k8s/net?k8s-version=\
  $(kubectl version | base64 | tr -d '\n')"

kubectl get pods -n kube-system
kubectl logs weave-net-xxxxx weave -n kube-system
```


## Network Loadbalancer



## Ingress


.
