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
