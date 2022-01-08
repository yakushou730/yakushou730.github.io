---
title: "helm"
authors: [yakushou730]
date: 2022-01-06T15:51:08+08:00
description: "helm"
tags: ["programming","k8s"]
draft: false
---

## What is Helm

一般來說，要 deploy 一個服務，即使是簡單的 app

也要建立一整套完整的 k8s 元件

ex: Secret, Deployment, Service, PersistentVolumeClaim... 等等，需要 apply 一堆 yaml 檔案

Helm 被稱為 k8s 的 package manager

> 想像一個遊戲由千百個檔案組成
> 
> 我們不需要自己處理那摩多檔案，透過安裝程式的方式
> 
> 做到一次安裝一個完整的遊戲，對應的檔案也都被安裝到正確的位置
> 
> 而 Helm 就像這樣，一次處理一堆 YAML 檔案和 K8s 物件，來設定 application

Helm 讓我們操作 k8s 的元件就像操作 app 一樣，而不是一堆檔案物件

```shell
# 透過 helm 安裝 wordpress
helm install wordpress
# 修改完 yaml 檔案套用
helm upgrade wordpress
# 可倒退版本
helm rollback wordpress
# 移除 wordpress
helm uninstall wordpress
# debug 資訊 
helm --debug
```

## Helm2 vs Helm3 
Helm3 多了
- RBAC (Role Based Access Control)
- CRD (Custom Resource Definition)
- 提供 snapshot (revision)

Helm3 移除了 Helm2 用的 Tiller

## Helm Components
Charts
- 一包 files
- 包含 Helm 需要的所有 instruction 在 k8s cluster 上建立物件
- 套用 Chart 到 cluster 上的時候，會建立 release (每個 release 都代表一次 chart 的安裝)
- 像 docker 有 docker hub 一樣，helm 也有對應的 online chart repository
  - ArtifaceHUB
- 為了追蹤 helm 對 k8s cluster 做了什麼變更，Helm 會儲存這些資訊 (as metadata)
  - Helm 把這資訊以 k8s secret 的方式存在 k8s cluster 
- yaml file 裡面可以像填空一樣放 template
  - 用來填空的資訊放在 values.yaml
- 我們可以下載到各式各樣的 Helm charts，之後的工作就是透過 values.yaml 填空把 configuration 設定好

```shell
# 安裝 chart
helm install [release name] [chart-name]
```

## Helm charts
一般的 helm chart 都還會包含的 files
- values.yaml: 用來放要嵌入 template 的資訊
- chart.yaml: 標明這份 chart 本身的資訊
  - 包含 apiVersion, appVersion, name...等等

chart.yaml
- apiVersion: 套用的 api 版本 (Helm3 用 v2)
- appVersion: 是指這個 app 要用哪個版本 deploy (ex: wordpress)
- version: 這份 chart 的版本
- name: chart 的名稱
- type:
  - application (default): 指是建立作為 deploying application
  - library: 指提供 utilities 來幫助 building charts
- dependencies: 有相依的套件，同樣以 helm 的方式載入
- keywords: 有點像 tag 的概念，可以被搜尋
- maintainers: 維護者資訊
- home: homepage url
- icon: icon url

![helm-structure.png](/k8s/helm-structure.png)

