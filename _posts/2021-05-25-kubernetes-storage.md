---
title: Kubernetes - Security
description:
search: false
categories:
  - kubernetes
tags:
  - kubernetes
permalink:

last_modified_at: 2021-05-26
---

Docker Storage local file system
- /var/lib/docker
  - aufs
  - containers
  - image
  - volumes


Layered architecture
1. Image layers (Read Only)
    - base OS Layer
    - apt packages
    - pip packages
    - source code
    - update entrypoint with "flask" command

2. Container Layer (Read Write)
    - container layer

what if we want to persist these data
we cloud add a persistent volume

```bash
docker volume create data_volume
```

- /var/lib/docker
  - aufs
  - containers
  - image
  - volumes
    - data_volume


1. volume mount

```bash
docker run -v data_volue:/var/lib/mysql mysql
```

2. bind mount

```bash
docker run -v /data/mysql:/var/lib/mysql mysql
```

```bash
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

- Storage drivers
  - AUFS (default)
  - ZFS
  - BTRFS
  - Device Mapper
  - Overlay
  - Overlay2

- Volume drivers
  - local
  - Azure File Storage
  - Convoy
  - DigitalOcean Block Storage
  - Flocker
  - gce-docker
  - GlusterFS
  - NetApp
  - RexRay
  - Portworx
  - VMware vSphere Storage

### Container Storage Interface (CSI)

> in the past, k8s used docker only for container runtime

### Container Storage Interface (CSI)

SHOULD call to provision/delete/place/decommission/make ....

### Container Network Interface (CNI)


## Volumes

Volume 을 붙여서 data를 permanently store

### Volumes & Mounts

```yaml
# local-volume-pod-definition.yaml

kind: Pod
apiVersion:
metadata:
spec:
  containers:
  - ...
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
    - name: data-volume
      hostPath:
        path: /data
        type: Directory
```

```yaml
# pvc-pod-definition.yaml

kind: Pod
apiVersion:
metadata:
spec:
  containers:
  - ...
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
    - name: data-volume
      persistentVolumeClaim:
        claimName: pv-claim
```

```yaml
# aws-ebs-volume-pod-definition.yaml

kind: Pod
apiVersion:
metadata:
spec:
  containers:
  - image:
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
    - name: data-volume
      awsElasticBlockStore:
        volumeID: <volume-id>
        fsType: ext4
```

## Persistent Volumes

Pod can use Persistent Volumes(PV)

```yaml
# pv-definition.yaml

kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```

```yaml
# pv-definition.yaml

kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

## Persistent Volume Claims

make the storage available to nodes

```yaml
# pvc-definition.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

```bash
kubectl create -f pvc-definition.yaml
kubectl get pvc
kubectl delete pvc myclaim
```

```yaml
1. persistentVolumeReclaimPolicy: Retain (default, remain until manually delete)
2. persistentVolumeReclaimPolicy: Delete (delete automatically)
3. persistentVolumeReclaimPolicy: Recycle (scrubbed to other claim)
```

## Configure Apps on Persistent Volumes


## Access Modes for Volumes


## Kubernetes Storage Object


## Storage Classes

1. Static Provisioning
  - last_modified_at
  - asdf
2. Dynamic Provisioning


```yaml
# storage-class-definition.yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard | pd-ssd
  replication-type: none | regional-pd
volumeBindingMode: WaitForFirstConsumer | Immediate
```


```yaml
# pvc-definition.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: some-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi
```


.
