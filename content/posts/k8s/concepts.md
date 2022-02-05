---
title: "concepts"
authors: [yakushou730]
date: 2021-12-28T14:35:13+08:00
description: "concepts"
tags: ["programming","k8s"]
draft: false
---

## 概念
**Container + Orchestration**

k8s 是一種容器編排技術，用於協調數百個對象的部署和管理 cluster 中的數千個容器

## VM vs Docker
- VM 是在 原本的 OS 上，再額外起來的完整的虛擬機器，具有完整的虛擬 OS
  - 每個 VM 裡面除了 OS 外還有各自的 libs 和 deps
  - 動輒以 GB 為單位
  - 啟動速度慢
- Docker 是基於原本的內核，起起來的 container
  - libs 和 deps 在各自容器內
  - 大約以 MB 為單位
  - 啟動速度快

## 術語
- node (minion)
  - 安裝了 k8s 的機器，container 會起在這上面
- cluster
  - 一組複數個 nodes ( 1 個 master / 多個 worker)
  - 會有一台 master node，負責監管和負責流程編排 node 上 的 container
- components
  - 安裝 k8s 時，會安裝的一些元件
    - API server
      - 可以作為 k8s 的前端 (系統操作入口)
      - 不管是使用者對 k8s 或是 node 對 master 都是透過 apiserver 溝通
      - 唯一可以和 etcd 溝通的元件，其他元件都必須透過 apiserver 才能訪問集群狀態
      - 以 RESTful interface 提供外部或內部組件使用
    - etcd
      - key-value store
      - 確保 master 間沒有 conflict
    - kubelet
      - 每個 node 都會安裝的 agent
      - 確保 container 可以運作在 node 上
    - container runtime
      - 用來跑 container 的環境 (ex: Docker)
    - controller manager
      - 負責 k8s cluster 故障檢測和恢復的自動化，執行多種 controller 
        - replication controller
          - 定期關聯 replication controller 和 pod，保證 pod 數量正確
        - node controller
          - kubelet 啟動的時候會通過 apiserver 註冊自己本身節點的資訊，定時向 apiserver 報告狀態
          - apiserver 收到後更新到 etcd
          - node controller 實現管理和監控 node 資訊
        - resourceQuota controller
          - 用來管理使用的資源不要超過設定
        - namespace controller
          - 用戶透過 apiserver 可以建立新的 namespace，存於 etcd
        - service account controller
          - 確保 default 的 ServiceAccount 在每個 namespace 中都存在
        - token controller
          - 用來監聽 service account 的建立/刪除 和 監聽 secret 的增加/刪除
        - service controller
          - 監聽 service 的變化
        - endpoint controller
          - endpoint 表示了 service 對應的所有 pod 副本的位址
          - 而 endpoint controller 是負責生成和維護所有 endpoints 的控制器，保證 service 到 pod 的對應總是最新的
      - orchestration 的大腦，在 nodes / containers 發生故障時發出通知和回應
      - 決定是否建立新的 container
    - scheduler
      - 分配 container 給 nodes
- master 上
  - 會安裝 kube-apiserver
  - 會安裝 etcd 儲存資訊
  - 會安裝 controller
  - 會安裝 scheduler
- worker 上
  - 會安裝 kubelet 和 master 溝通

## k8s component 的 kind 種類
**ConfigMap**

指 系統相關設定的 key-value pair

讓 app pod 和 設定 分開，達到 decouple
- pod 需要 configMap 的時候再掛載來用就好

**CronJob**

K8s 的排程

> k8s 中最小單位是 pod
> 
> 所以
> 
> CronJob 在使用的時候也會產生對應的 pod

**Deployment**

Deployment 負責管理 ReplicaSet 和 Pod 的更新

**Service**

Service 是用來建立一個網路連線可以連到 pod

**Secret**

把敏感資訊以 `非明碼的方式` 放在 k8s 上
- 資料庫帳密
- access token
- ssh key...等等

k8s 可以把 secrets 當作環境變數來使用

**Log**
查看 pod 的 log
```shell
# l flag => label
# f flag => stream pod logs (stdout)
kubectl logs -l app=service --all-containers=true -f --tail=100
```
