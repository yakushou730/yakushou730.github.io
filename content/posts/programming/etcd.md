---
title: "etcd"
authors: [yakushou730]
date: 2022-03-06T11:39:08+08:00
description: "etcd"
tags: ["programming","git"]
draft: false
---

## install
```shell
brew install etcd
```

可在 service list 確認是否正常
```shell
brew services list

# results
etcd               started yakushou730 ~/Library/LaunchAgents/homebrew.mxcl.etcd.plist
```

## 操作
安裝完後系統會有 etcdctl 的指令可以使用

寫入
```shell
etcdctl put foo bar
```

讀取
```shell
etcdctl get foo
# result
foo
bar


# 另一個範例
etcdctl get product.rpc --prefix
# result
product.rpc/7587860940551895303
192.168.0.221:8081
```

刪除
```shell
etcdctl del foo
# result
1
```
