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



