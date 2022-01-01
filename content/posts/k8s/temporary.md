---
title: "temporary"
authors: [yakushou730]
date: 2021-12-24T11:28:29+08:00
description: "temporary"
tags: ["programming","k8s"]
draft: false
---

## 從 AWS EKS 下載 k8s config
範例:
```shell
aws eks --region ap-southeast-1 update-kubeconfig --name <cluster-name> --kubeconfig <the name that we download to local> 
```

## 載下來的 kube config 如何載入
載下來的 config 可以放到 `~/.kube` 內

然後在 bash profile 設定環境變數 `KUBECONFIG`

```shell
# k8s config setting
export KUBECONFIG=~/.kube/service-prod-config:\
~/.kube/service-staging-config
```

## 使用 context
```shell
# 取得 context list 
kubectl config get-contexts
# 使用特定 context
kubectl config use-context <context name>
# 詢問當前 context
kubectl config current-context
```

## 使用 kubectx
可快速切換 k8s context 的工具
```shell
# 切到 <target context>
kubectx <target context>
# 列出所有 contexts
kubectx
# 切換到上一次使用的 context
kubectx -

# 設定別名 和 使用別名
kubectx <alias context name>=<original context name>
kubectx <alias context name>
```

## 進入 pod
範例:
進入 pod 並執行 sh 指令
```shell
kubectl exec -it <pod name> -- sh
```

## 顯示 cluster 資訊
```shell
kubectl cluster-info
```

## 取得 node 資訊
```shell
# -o wide 是 output in the plain-text format with any additional information.
kubectl get nodes -o wide
```

## 查看 k8s 版本資訊
```shell
kubectl version
```

## 查看 node 資訊
```shell
kubectl describe nodes <node name>
```

## PODs
- container 不會直接被放到 node 內
- 是透過封裝在 pod 裡面才被放到 node
- 一個 pod 是一個單一的 application (通常是一對一)
  - 但還是可以在一個 pod 裡面放多個 container (一起建立/一起銷毀)
- pod 是在 k8s 內可以建立的最小單位
- 負載量過大的時候會以 pod 為單位創建新的 application

指令:
```shell
# 從 dockerhub 拉下 nginx image 並執行成 名為 nginx 的 pod
kubectl run nginx --image nginx
# 取得 pods 列表
kubectl get pods
# 取得特定 pod 的詳細描述
kubectl describe pod <pod name>
# 直接連同 deployment 一起建立
kubectl create deployment nginx --image=nginx
# 刪除 pod
kubectl delete pod <pod name>
# 透過檔案建立 pod
kubectl create -f <file name>
# 透過基本旗標建立 pod yaml
kubectl run <container name> --image=<image name> --dry-run=client -o yaml > <taget yaml file name> 
# 套用 yaml 檔案
kubectl apply -f <file name>
```

## Replications controller & ReplicaSets
讓 pods 可以執行特定數量，如果掛掉的話可以重啟回固定數量

有點像是把 Pod 包起來的概念，Replication controller 會固定在內部的 pod 數量

- Scaling
  - Replication Controller 支援跨 nodes，即可以控管多數個 nodes 總共需要幾個固定數量的 pods
- Load Balancing
  - Replication Controller 也做 Load balancing，把流量導去不同的 pods

ReplicaSet 要取代舊式的 Replication Controller

## Deployments
把新版本的 image 部署到 node 上

可以做 rolling update / rollback

Deployment 可以視為更大的範圍(集合)，包含了 Replica Set 在裡面

## Deployment Updates and Rollback
建立 deployment 的時候，會觸發 rollout

一個新的 rollout 會建立一個新的 deployment revision

(例: 第一次發布為 revision 1，第二次發布為 revision 2)

revision 可以作為進版或退版的版本

指令:
```shell
# 查看特定 deployment 的 rollout 狀態
kubectl rollout status deployment/<deployment name>
# 查看特定 deployment 的 rollout 歷史紀錄
kubectl rollout history deployment/<deployment name>
```

- Deployment strategy
  - recreate
    - 舊版全下 接著新版全上
    - 會有 downtime
  - rolling update (default)
    - 舊的下一個 接著上一個新的
    - 一個一個更新
    - 實際是會產生一個新的 ReplicaSet，然後舊的 ReplicaSet下架舊的 Pod，並在新的 ReplicaSet 上架新的 Pod

```shell
# 使用新的 yaml 檔案來觸發新的 rollout
# 且建立新版的 deployment
kubectl apply -f deployment-definition.yaml
# 直接變更 image (會導致 yaml 和實際的物件對不起來)
kubectl get image deployment/<deployment name> <container name>=<new image name>
# rollback
kubectl rollout undo deployment/<deployment name>
# 如果要在 rollout history 顯示 CHANGE-CAUSE 的話，要補 record flag
kubectl create -f <deployment file name> --record
```

## Network 101
- 每個 Pod 都會被給予一個 IP address
  - 但是要避免 private IP conflict (要自己另外處理)
- k8s 預期我們需要自己完成的項目
  - 所有的 container / pod 都不透過 NAT 達到互相溝通
  - 所有的 nodes 也可以和 containers 不透過 NAT 達到互相溝通
  - 可以使用的 solution 套件
    - Cisco ACI network
    - Cilium
    - Big Cloud Fabric
    - Flannel
    - VMWare NSX-T
    - Calico

## Service
k8s 的 service 用來協調 外部和內部 application 的溝通

- Type
  - NoePort
  - ClusterIP
  - LoadBalancer


**NodePort**
- 意思是在 Node上面開一個 Port，可以從外部對應到內部的 pod
- 可以設定 mapping 給 Node port 對 Pod port
- 以 service 的觀點來看的對應
  - service 就像是一個 node 內的 virtual server，在 cluster 內有他自己的 IP address `ClusterIP` 
  - Pod 上面預期 request 進入的 port 稱為 `TargetPort`
  - Service 上對應給 TargetPort 的 port 稱為 `Port`
  - 開在 Node 上的 port 稱為 `NodePort`，有範圍限制，default 為 30000 ~ 32767 
- 在相同的 node 內的相同 pod，因為 label 一樣，所以 service 會隨機導進流量
- 不同 node 的話，開的 node port 設定一樣的話，service 仍可以共用

**ClusterIP**
- 在 Cluster 內建立一個 virtual IP，讓內部的 pods 間可以溝通
- 比如說有一個 backend group 裡面都是 backend pod，以及一個 frontend group 裡面都是 frontend pots

  在這兩個 group 之間，需要透過 ClusterIP service 來做通訊

**LoadBalancer**
- 對 application 提供 Load Balancer
- LoadBalancer 是掛在 Cluster 等級，用來接外部的流量要進哪台 Node
- 大部分的 Cloud provider 都有提供這功能

# 範例
![k8s-docker-architecture-sample](/k8s/k8s-docker-architecture-sample.jpg)
