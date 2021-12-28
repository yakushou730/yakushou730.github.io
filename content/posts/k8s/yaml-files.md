---
title: "YAML Files"
authors: [yakushou730]
date: 2021-12-28T19:02:38+08:00
description: "YAML Files"
tags: ["programming","k8s"]
draft: false
---

## General YAML template
必要的 4 個欄位
```yaml
apiVersion: # 要用來建立物件的 k8s API 版本 
kind: # 要建立的物件
metadata: # 物件的其他資訊，如 name, labels...

spec: # 建立物件需要的訊息

```
- apiVersion (string)
  - POD: v1
  - Service: v1
  - ReplicaSet: apps/v1
  - Deployment: apps/v1
- kind (string)
  - Pod
  - Service
  - ReplicaSet
  - Deployment
- metadata (dictionary)
  - name
  - labels
    - labels 下的 key/value 值是可以自訂的
- spec (dictionary)
- 

## POD with YAML
pod-definition.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```
指令
```shell
# 建立
kubectl create -f pod-definition.yaml
# list
kubectl get pods
kubectl get po
```

## Replication Controller with YAML
rc-definition.yaml
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template: # 下面是放 POD 的 meta data 和 spec，注意縮排
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx 
  replicas: 3
```
指令
```shell
kubectl apply -f rc-definition.yaml
# 取得 replication controllers
kubectl get replicationcontroller
```

## ReplicaSet with YAML
和 replication controller 最主要的差別是 selector

replicaset-definition.yaml
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template: # 下面是放 POD 的 meta data 和 spec，注意縮排
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```
指令
```shell
# 建立
kubectl create -f replicaset-definition.yaml
# 更新 replicas 數量可以怎麼做
kubectl replace -f replicaset-definition.yaml # 要事先更新好 yaml
# or
kubectl scale --replicas=<target number> -f replicaset-definition.yaml
# or
kubectl scale --replicas=<target number> replicaset <replicaset name>
# 取得 replica sets
kubectl get replicaset
kubectl get rs
# 刪除 replicaset (底下的 pods 也會一並刪除)
kubectl delete replicaset <replicaset name>
# 直接更改物件的值 (會開啟 vim)
kubectl edit replicaset <replicaset name>
```

## Deployment with YAML
內容和 replica set 非常像

deployment-definition.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template: # 下面是放 POD 的 meta data 和 spec，注意縮排
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```
指令
```shell
# 建立
kubectl create -f deployment-definition.yaml
# list
kubectl get deployments
# 因為 deployment 會自動建立 replicaset，所以可以順便看一下
kubectl get replicaset
kubectl get pods
# 列出全部項目
kubectl get all
```
