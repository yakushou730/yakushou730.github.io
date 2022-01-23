---
title: "Ultimate Service V3"
authors: [yakushou730]
date: 2022-01-23T21:41:43+08:00
description: "Ultimate Service V3"
tags: ["programming","golang"]
draft: false
---

# Modules

Project:
- a repo of code
- defines the philosophy, policy and guideline

```shell
go mod init [module name]
# 範例
go mod init github.com/ardanlabs/service3-video
```

```shell
go env
#
GOMODCACHE="指向 module cache"
GOPROXY="proxy的位址和方式"
GONOPROXY="這個位址的不要透過proxy，表示直接抓這個位址"

# 清掉 module cache
go clean -modcache

# 自動處理專案的 import
go mod tidy
```

go.mod 會存放這個專案會用到的 dependency module

go.sum 會存放用來驗證要用到的 dependency module 的編碼

每當 proxy 抓到新版時，就會另外算出 checksum 放在 checksum DB，讓之後其他人抓 module 的時候可以比對是不是正確的 module

```shell
# 建立 vendor 資料夾並把相依抓下來放到這個目錄下
# 這樣專案就會直接找這個資料夾下的資料
go mod vendor
```

> 如果發生抓下來的 module 出現 incompatible 怎麼辦?
> 
> 因為 module 其實還額外支援 module 版本，所以可能是要指定特殊版本來使用
> 
> 範例: [github.com/dimfeld/httptreemux/v5](https://linuxhint.com/bash_eval_command/)
>
> [Reference](https://github.com/dimfeld/httptreemux/blob/v5.3.0/go.mod)
> 
> 上面的連結可以看出 tag v5.3.0 的版本號是給 v5 用的
> 
> 這邊的 v5 是指說要抓這個特殊版本

MVS: minimal version selection

如果同時有 module A 相依 module C v1.4，和 module B 相依 module C v1.5

則系統會抓 C v1.5
