---
title: Kubernetes - Application Lifecycle Management
description:
search: false
categories:
  - kubernetes
tags:
  - kubernetes
permalink:

last_modified_at: 2021-05-04
---


## Rolling Updates and Rollbacks in Deployments

### Rollout and Versioning

```
kubectl rollout status <deployment-name>
kubectl history hisyory <deployment-name>
```

### Deployment Strategy

#### Recreated Strategy
> 한 번에 버젼업 앱 생성

```
kubectl describe deployment <deployment-name>

scale 0 >> scale 5
```

#### Rolling Updates (default)
> 한 앱씩 버젼업 (seamless)

```
kubectl describe deployment <deployment-name>

scale 0 >> scale 1
scale 1 >> scale 2
...
```

```
kubectl get replicasets
kubectl rollout undo <deployment-name>
kubectl get replicasets
```

```
commands for Rolling Update

kubectl create -f <deployment-yaml>
kubectl get deployments
kubectl apply -f <deployment-yaml>
kubectl set image <deployment-name> <version (nginx=nginx:1.9.1)>
kubectl rollout status <deployment-name>
kubectl rollout history <deployment-name>
kubectl rollout undo <deployment-name>
```

## Configure Applications

### Commands and Arguments

```
FROM Ubuntu
CMD ["sleep 5"] (X)
CMD ["sleep", "5"] (O)

docker run ubuntu-sleeper sleep 10
```

```
FROM Ubuntu
ENTRYPOINT ["sleep"]

docker run ubuntu-sleeper 10
```

```
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]

docker run ubuntu-sleeper >> sleep 5
docker run ubuntu-sleeper 20 >> sleep 20
docker run ubuntu-sleeper --entrypoint sleep2.0 20 >> sleep2.0 20
```

```yaml
pod-command-args-definition.yaml

apiVersion:
kind: Pod
metadata:
spec:
  containers:
    - name:
      image:
      command: ["sleep2.0"]     (ENTRYPOINT)
      args: ["10"]              (CMD)
```


### ENV variable types

```yaml
pod-command-args-definition.yaml

apiVersion:
kind: Pod
metadata:
spec:
  containers:
    - name:
      image:
      ports:
        - containerPort: 8080
  env:
    - name: APP_COLOR           (Direct Plain Key Value)
      value: pink

    - name: APP_COLOR           (ConfigMap)
      valueFrom:
        configMapKeyRef:

    - name: APP_COLOR           (Secrets)
      valueFrom:
        secretKeyRef:
```

#### ConfigMaps

```
imperative

kubectl create configmap <config-name> \
    --from-literal=<key>=<value>

kubectl create configmap <config-name> \
    --from-file=<file-path>
```

```
app-config

APP_COLOR: blue
APP_MODE: prod
```


```yaml
declarative
kubectl create -f <.>

config-map-definition.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
spec:
  APP_COLOR: blue
  APP_MODE: prod
```


```
kubectl get configmaps
kubectl describe configmaps
```


```yaml
pod-configmap-definition.yaml

spec:
  containers:
    - name:
      image:
      envFrom:
        - configMapRef:
            name: app-config
```

#### Secrets

```
imperative
kubectl create secret generic <secret-name> \
    --from-literal=<key>=<value>

kubectl create configmap <config-name> \
    --from-file=<file-path>
```

```yaml
secret-data.yaml

apiVersion:
kind: Secret
metadata:
    name: app-secret
data:
  DB_HOST: hXlzcWw=       (echo -n 'mysql' | base64)
  DB_USER: cm9vdA==       (echo -n 'root' | base64)
                          (echo -n 'root' | base64 --decode)
```

```yaml
pod-secret-definition.yaml

spec:
  containers:
    - name:
      image:
      envFrom:
        - secretRef:
            name: app-secret
```


## Scale Applications

> check previous topic 'Deployments - Rolling updates and Rollback'

### Multi Container PODs

> 3 design patterns
1. Sidecar
2. Adapter
3. Ambassador

#### initContainers

> Process in the initContainer must run to a completion before the real container hosting the application starts

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

> If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```



## Self-Healing Application

> check previous topic 'Replicasets and Replication Controllers  

> The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.


.
