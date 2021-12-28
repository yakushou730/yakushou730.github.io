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
      - 可以作為 k8s 的前端
    - etcd
      - key-value store
      - 確保 master 間沒有 conflict
    - kubelet
      - 每個 node 都會安裝的 agent
      - 確保 container 可以運作在 node 上
    - container runtime
      - 用來跑 container 的環境 (ex: Docker)
    - controller
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