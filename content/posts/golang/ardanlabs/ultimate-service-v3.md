---
title: "Ultimate Service V3"
authors: [yakushou730]
date: 2022-01-23T21:41:43+08:00
description: "Ultimate Service V3"
tags: ["programming","golang"]
draft: false
---

[**Github repo**](https://github.com/yakushou730/ultimate-service-v3)

整理了下，如果整個 cluster 重啟，要跑哪些步驟
```shell
# 把 cluster shutdown
make kind-down

# 1. 啟動 kind
make kind-up

# 2. build 新的 service image
make all

# 3. 設定 kind 要抓本機的 image
make kind-load

# 4. 設定 kind cluster 要吃本機的 config 設定
make kind-apply

# extra: 看 log
make kind-logs

# extra: 直接更新 kind 設定
# all + kind-load + kind-restart
make kind-update
```

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

## Kubernetes

```go
package main

import "log"

var build = "development"

func main() {
	log.Println("starting service", build)
}
```

build 的時候 可以強行指定抽換 var
```shell
# -ldflags 表示會發生參數抽換
# -X 表示鏈結選項 格式 -X [importpath].[name]=[value]
go build -ldflags "-X main.build=local"


# 清除 build 出來的執行檔
go clean
```
這樣會印出 `starting service local`

參數可以參閱 [Go build flags](https://pkg.go.dev/cmd/go#hdr-Compile_packages_and_dependencies)

`dockerfile`
```dockerfile
FROM golang:1.17 as build_sales-api
# CGO_ENABLED 0 才可以做到交叉編譯
# 即編譯成不同的 os 和 architect
ENV CGO_ENABLED 0
# ARG 是可以從外部帶入的參數
ARG BUILD_REF

# Copy the source code into the container
COPY . /service

# Build the service binary
WORKDIR /service
RUN go build -ldflags "-X main.build=${BUILD_REF}"

# Run the Go Binary in alpine
FROM alpine:3.15
ARG BUILD_DATE
ARG BUILD_REF
COPY --from=build_sales-api /service /service/service
WORKDIR /service
CMD ["./service"]

LABEL org.opencontainers.image.created="${BUILD_DATE}" \
      org.opencontainers.image.title="sales-api" \
      org.opencontainers.image.authors="Shou Tseng <yakushou730@gmail.com>" \
      org.opencontainers.image.source="https://github.com/yakushou730/service/app/sales-api" \
      org.opencontainers.image.version="${BUILD_REF}" \
      org.opencontainers.image.vendor="Yakushou730"

```

`Makefile`
```makefile
SHELL := /bin/bash

run:
	go run main.go

build:
	go build -ldflags "-X main.build=local"

# ============================================================
# Building containers
VERSION := 1.0

all: service

service:
	docker build \
		-f zarf/docker/dockerfile \
		-t service-amd64:${VERSION} \
		--build-arg BUILD_REF=${VERSION} \
		--build-arg BUILD_DATE=`date -u +"%Y-%m%dT%H:%M:%SZ"` \
		.

```

> 跳過 kind 設定 local k8s 的段落

- 稍微列一下注意事項
  - 在本機開發安裝 kind 來模擬 k8s 環境
  - 注意 golang 專案檔名，或是確保 build 出來的 binary 是自己要的，不然包 docker run 會失敗
  - 確認 kind load 有設定，這樣才會抓本機的 docker images
  - kubectl 要確認 namespace，不然會查不到已存在的 k8s component

- kustomization 的用法是 可以 patch 原本的 base k8s yaml
  - 如 有一個 `base-service.yaml` 內含 deployment 資訊，透過 kustomization 可以把額外的資訊當成補丁更新上去

> 如果 go mode tidy 跳了錯誤是關於
> 
> module declares its path as: go.uber.org/automaxprocs
> 
> but was required as: github.com/uber-go/automaxprocs
> 
> 表示該 go module 的路徑不是照 github 的規範來做，就要修改成 go module 該套件定義的路徑

## Initial Service Design
5 個資料夾分類

- app
  - services
    - metrics `sidecar for service-api`
    - service-api
      - handlers
      - tests
      - main.go
  - tooling
    - logfmt
      - main.go
    - service-admin
      - commands
      - main.go
- business `business logic`
  - core
    - report
  - data
    - schema
    - store
      - product `store 內的東西彼此要獨立 不要互相 import`
      - user
    - tests
  - sys `system oriented`
    - auth
    - database
    - metrics
    - validate
  - web
- foundation `是個 reusable 的項目，彼此間可以 import 引用`
  - docker
  - keystore
  - logger
  - web
  - worker
- vendor `3rd party`
- zarf `configuration`

```shell
# 清理 docker 資源
docker system prune
# This will remove:
# all stopped containers
# all networks not used by at least one container
# all dangling images
# all build cache
```

**如果發生應該要 顯示出來的欄位而沒出現，先猜可能是大小寫問題造成的 private / public 問題**

> 處理 logger，可以先在 main.go 做，不急著拉去 package
> 
> 除了 main.go 不應該有其他 package 可以存取 configuration

新專案的處理順序
1. 定義 project layout
2. 建立 logger
3. 建立 configuration
4. 建立 debugging / metrics support
5. 建立 shutdown signaling and load shedding

log 用到的 package [go.uber.org/zap](https://github.com/uber-go/zap)

可以視覺化 pprof 的套件 [https://github.com/divan/expvarmon](https://github.com/divan/expvarmon)

> 一般來說，linter 會建議 package 上方要有註解，此時可以在那個 package 專門建立一個 doc.go 在 package 上方做註解，不需要其他程式碼

## HTTP Routing Basis
建立可以讓 k8s 偵測的 readiness / liveness endpoint

## Web Framework
設計呼叫流程，包含怎麼套用 middleware

呼叫 middleware 就像洋蔥圈的方式，最內層是要呼叫的 function

由外往內一層一層進去

## Middleware
errors
- Is: error variable comparison
- As: type comparison

middleware 的 layer 順序
1. (最外層) Logger
2. Error
3. panics
4. (最內層) Handler action

