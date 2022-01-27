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


