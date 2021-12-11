---
title: "Makefile"
authors: [yakushou730]
date: 2021-12-05T21:01:05+08:00
description: "Makefile"
tags: ["programming"]
draft: false
---

範例

```makefile
GREENLIGHT_DB_DSN='postgres://greenlight:@localhost/greenlight?sslmode=disable'

## help: print this help message
.PHONY: help
help:
	@echo 'Usage:'
	@sed -n 's/^##//p' ${MAKEFILE_LIST} | column -t -s ':' | sed -e 's/^/ /'

.PHONY: confirm
confirm:
	@echo 'Are you sure? [y/N] ' && read ans && [ $${ans:-N} = y ]

## run/api: run the cmd/api application
.PHONY: run/api
run/api:
	go run ./cmd/api

## db/psql: connect to the database using psql
.PHONY: db/psql
db/psql:
	psql ${GREENLIGHT_DB_DSN}

## db/migrations/new name=$1: create a new database migration
.PHONY: db/migrations/new
db/migrations/new:
	@echo 'Creating migration files for ${name}...'
	migrate create -seq -ext=.sql -dir=./migrations ${name}

## db/migrations/up: apply all up database migrations
.PHONY: db/migrations/up
db/migrations/up: confirm
	@echo 'Running up migrations...'
	migrate -path ./migrations -database ${GREENLIGHT_DB_DSN} up

```

- @ 前綴可以避免印出那行指令
- namespace 用 `/` 來分的話，可以有按 tab 的 autocomplete 可以用

  但這種做法有個問題，因為 makefile 原本的用意是產生檔案

  所以如果碰到對應路徑的檔案已經存在，則會停止執行指令
- 可以用 .PHONY 避免前一點的問題
- confirm 在這邊的用法是 prerequisite target
- 只輸入 make 的話，預設是執行第一條指令
- 可以用 include 加入參數檔案
