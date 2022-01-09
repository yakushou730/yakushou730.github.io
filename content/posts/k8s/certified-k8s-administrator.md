---
title: "Certified K8s Administrator"
authors: [yakushou730]
date: 2022-01-09T21:07:35+08:00
description: "Certified K8s Administrator"
tags: ["programming","k8s"]
draft: false
---

## Core Concepts

> k8s 的目的是要以自動化的形式管理 container 形式的 app
> 
> 讓使用者可以簡單的部署，並讓各服務的溝通可以簡單化

解說:

假設現在有兩艘船，cargo ship 和 control ship

- Cargo Ship
  - 負責實際運載 container 的工作
  - 每艘船的船長，負責管理船上的 activities，負責接收當 master 需要裝載 container 時的資訊，
    並回報給 master 這艘船的狀態和 container 的狀態
- Control Ship
  - 負責監測和管理 cargo ships
  - 透過 control plane 元件負責管理 cargo ship 的大小事
  - ETCD: 每天都有很多 container 要被載到 cargo ships 上，所以需要紀錄不同的船上有哪個 container，包含載入的時間
    而這些是以 key-value 的形式被保存的
  - kube-scheduler: 當 cargo ship 來的時候，要負責判斷並把對應的 container 奘載給他
  - Controller manager:
    - 指派特殊工作，例: operation team 要處理交通控制，或是 container 壞掉的時候，怎麼樣建立一個可以用的 container
    - service office 用來讓各部門溝通，或是讓不同的船之間溝通
- 最高部門: kube-apiserver，各部門之間管理的最高層級，負責 Cluster 的工作協調

對應到 k8s 的名詞
- Worker Nodes: host Application as Containers
  - Container Runtime Engine: 讓 node 可以處理 container
  - kubelet: 船長的角色，是一個裝在 node 上的 agent，接收來自 kube-apiserver 的指令，在 node 上 deploy 或 destroy container
  - kube-proxy: 讓 nodes 之間可以知道要溝通的服務在哪邊 (web container 對應 db container)
- Master: Manage, Plan, Schedule, Monitor Nodes
  - ETCD Cluster: 一個 key-value 的資料庫
  - kube-scheduler: 負責把 container 起在對應的 node 上
    - 可能包含的條件有 resource requirements for nodes capacity, policies, ...etc
  - Controller manager
    - Node-Controller: 處理新增/刪除 node，以及處理出問題的 node
    - Replication-Controller: 確保期望的 container 數量都是對的
  - kube-apiserver: 管理 cluster 的最高層級，提供 k8s API 來讓使用者操作
    定時監控 kubelet 上 node 和 container 的狀態報告 

## ETCD

> ETCD is a distributed reliable key-value store that is simple, secure & fast

default port: 2379

```shell
# 安裝完後執行 binary 檔案
./etcd
# 設定 key-value
./etcdctl set key1 value1
# 取得 value
./etcdctl get key1 # 得到 value1
# help
./etcdctl
```

所有 k8s 上要紀錄的資料都會更新在 etcd datastore 上，然後才會在 cluster 上生效

不同的安裝方式
1. 自己獨立安裝的，把 binary 檔案設定成 service 並執行
   - 需要注意的欄位 `--advertise-client-urls https://${INTERNAL_IP}:2379`，這是要設定在 apiserver 上的 url
2. 透過 kubeadm 安裝的
   - 會把 etcd 做成 pod 安裝在 cluster 上 kube-system 的 namespace 中
   - 所以可以進到 pod 裡面並操作 key-value store
   - 多 cluster 的情況，要注意的欄位是 `--initial-cluster controller-0=https://${CONTROLLER0_IP}:2380,controller-1=https://${CONTROLLER1_IP}:2380`
```shell
kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```

