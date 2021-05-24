---
title: Kubernetes - Security
description:
search: false
categories:
  - kubernetes
tags:
  - kubernetes
permalink:

last_modified_at: 2021-05-07
---

> solve certificate issues


## Kubernetes Security Primitives

```

### Secure Hosts

> password based authentication disabled  
SSH key based authentication  

### Secure Kubernetes

#### Authentication

> who can access to kube-apiserver is **IMPORTANT**

#### Authorization

> what can the do ?

TLS Certificates between various components

```


## Secure Persistent Key Value Store


## Authorization

### Auth Mechanisms - Basic

> not recommended, but the easiest  
Setup Role Based Auth for the new users


## Authentication

> secure access to k8s cluster

admins, developers, users,

2. bots(service accounts)
  - create serviceaccount  
  - ~~CKAD part~~

```
curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"
```

static password file
static token file
--token-auth-file=user-details.csv


## Security Contexts

- ??

## TLS Certificates for Cluster Components

> prerequsition - TLS Basic

CA (certificate authority): public/private key keeper

1. server Cert. for servers
  - kube-apiserver: apiserver.crt, apiserver.key  (main, gets all info)
  - etcd-server: etcdserver.crt, etcdserver.key
  - kubelet-server: ...


2. client Cert. for clients
  - admin: admin.crt, admin.key
    - kubectl REST API
  - kube-controller-manager: ...
  - kube-scheduler: ...
  - kube-proxy: ...
  - kube-apiserver: apiserver-kubelet-client.crt, apiserver-kubelet-client.key
  - kube-apiserver: apiserver-etcd-client.crt, apiserver-etcd-client.key
  - kubelet-server: kubelet-client.crt, kubelet-client.key


how to generate for the cluster


```bash
**CA**

// Generate Keys
openssl genrsa -out ca.key 2048
>> ca.key

// Ceritificate Signing Request
openssl req -new -key ca.key -subj "/CN=KUBERNETS-CA" -out ca.csr
>> ca.csr

//Sign Certificates
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
>> ca.crt

//now CA has ca.key/ca.csr/ca.crt
```

```bash
**Admin user**

// Generate Keys
openssl genrsa -out admin.key 2048
>> admin.key

// Ceritificate Signing Request
openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
>> admin.csr

//Sign Certificates
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
>> admin.crt

//now SYSTEM:MASTERS group has admin.key/admin.csr/admin.crt
```

```bash
curl https://kube-apiserver:6443/api/v1/pods \
    --key admin.key --cert admin.crt \
    --cacert ca.crt
```

```bash
docker ps -a
docker logs <container-id>
```


## Certificates API

CA is just pair of files
only on that server
control node is also CA server
build-in certificates API in control node
CertificateSigningRequest object from control node
  - review requests
  - apporve requests

```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  signerName: kubernetes.io/kube-apiserver-client
  groups:
  - system:authenticated
  request: <some-key>
  usages:
  - digital signature
  - key encipherment
  - server auth
```

```
kubectl get csr
kubectl certificate approve <csr-name>
kubectl certificate deny <csr-name>
kubectl delete csr <csr-name>
kubectl get csr jane -o yaml
```

certificate related operations are in kube-controller-manager
including csr-manager, <?>

```
cat /etc/kubernetes/manifests/kube-controller-manager.yaml

--cluster-sining-cer-file
--culster-signing-key-file
```

### KubeConfig

```
curl https://<some-url:port>/api/vi/pods \
--key admin.key
    --cert admin.crt
    -- cacert ca.crt
```



```
$HOME/.kube/config
```

Config files contains 3 types

1. Clusters
    - Dev
    - Prod
    - Google

  ```
  --server <some-url>:<port>
  ```

2. Contexts
    - admin@Prod
    - dev@Google

3. Users
    - admin
    - dev
    - prod

```
--client
--client
--so,ethid
```


```yaml
# kubeconfig-deifinition.yaml

apiVersion: v1
kind: Config
clusters:
- name: Prod
  cluster:
    certificate-authority: /etc/kuberenetes/pki/ca.crt
    certificate-authority-data: <raw encoded text for ca.crt>
    server: http://some-url:port
- name:
  ...
- name:
  ...

contexts:
  - name:
    context:
      cluster:
      user:
      namespace:
  - name:
  - name:

users:
- name:
  user:
    client-certificate: admin.crt
    client-key: admin.key
- name:
  ...
- name:
  ...
```

```
kubectl config view --kubeconfig=<>
```


### API Groups

```
curl https://kube-master:6443/version
curl https://kube-master:6443/api/v1/pods
curl https://kube-master:6443 -k    # list available api groups
```

### Authorization

ABAC (Authroization Based Access Control)

RBAC (Role Based Access Control)

Webhook

Node

AlwaysAllow

AlwaysDeny


```
--authorization-mode=AlwaysAllow           (default)
--authorization-mode=Node, RBAC, Webhook
```

#### RBAC

```yaml
# developer-role-definition.yaml

apiVersion: rbac.authroization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list", "get", "create", "update", "delete"]
    resourceNames: ["blue", "orange"]
  - apiGroups: [""]
    resources: ["ConfigMap"]
    verbs: ["Create"]
```

```yaml
# devuser-developer-binding.yaml

apiVersion: rbac.authorazation.k8s.io/v1
kind: RoleBinding
metadata:
  name: devsuser-developler-binding
subjects:
  - kind: user
    name: dev-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl create -f dev-user-developer-binding.yaml

kubectl get roles
kubectl get rolebindings
kubectl describe role developer
kubectl describe rolebinding dev-user-devloperuser-binding
```

```bash
# check access

kubectl auth can-i create deployments
kubectl auth can-i delete nodes

kubectl auth can-i create deployments --as dev-user
kubectl auth can-i delete nodes --as dev-user
```

### Role Bindings and Cluster Roles

> resources are categorized as either namespace or cluster

#### Role Bindings (Namespaced)

> Role, RoleBinding, PVC, pods, deployments ...

#### Cluster Roles (Cluster scoped)

> ClusterRole, ClusterRoleBinding, PV, namespaces, nodes ...

```yaml
# cluster-admin-role.yaml

apiVersion: rbac.authorazation.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
  - apiGroups: [""]
    resources: [""]
    verbs: ["list", "get", "create", "delete"]
```

```bash
kubectl create -f cluster-admin-role.yaml
```

```yaml
# cluster-admin-role-binding.yaml

apiVersion: rbac.authorazation.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
  - kind: User
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

```
kubectl create -f cluster-admin-role-binding.yaml
```


## Images Securely

```yaml
# some-definition.yaml

...
spec:
  containers:
    - name:
      image: docker.io/nginx/nginx
```

### Private Repository

```yaml
# some-definition.yaml

...
spec:
  containers:
    - name:
      image: private-registry.io/apps/internal-app
```

## Security Context

```yaml
**pod-security-context-definition.yaml**

...
spec:
  conatiners:
    - name:
      image:
      command:
        securityContext:          (supported at the container level)
          runAsUser: 1000
          capabilities:
            add: ["MAC_ADMIN"]
```

## Network Policies

### Ingress & Egress

> All Allow (default)

web pod -- api pod -- db pod
1. web pod
  - allow ingress traffic from user on port 80
2. api pod
3. db pod
  - allow ingress traffic from api pod on port 3306


```yaml
**network-policy-definition.yaml**

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
    policyTypes:
      - Ingress
    ingress:
      - from:
        - podSelector:
            matchLabels:
              role: api-pod
        ports:
          - protocol: TCP
            port: 3306
```

support network policies : Kube-router, Calico, Romana, Weave-net  
NOT support network policies : Flannel




.
