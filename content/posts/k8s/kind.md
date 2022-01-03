---
title: "kind"
authors: [yakushou730]
date: 2022-01-03T11:55:15+08:00
description: "kind"
tags: ["programming","k8s"]
draft: false
---

[kind official page](https://kind.sigs.k8s.io/docs/user/quick-start/)

kind 可以用來在 local 建立 k8s cluster

```shell
# 安裝 kind
brew install kind
# 建立 cluster
kind create cluster
# 查看 clusters
kind get clusters
# 查看 kind 的 kubeconfig
kind get kubeconfig
# 切換到 kind cluster
kubectl config use-context kind-kind
#  查看 cluster 狀態
kubectl cluster-info
```
