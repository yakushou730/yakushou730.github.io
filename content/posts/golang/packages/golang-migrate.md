---
title: "golang-migrate"
authors: [yakushou730]
date: 2021-11-21T23:36:24+08:00
description: "golang-migrate"
tags: ["golang"]
draft: false
---

[**github link**](https://github.com/golang-migrate/migrate)

以 postgreSQL 為例

## 安裝 (用 homebrew)
```shell
$ brew install golang-migrate
```

## 確認版本
```shell
$ migrate -version
```

## 建立 migration 檔案
```shell
# 建立 migration 檔案
$ migrate create -seq -ext=.sql -dir=./migrations create_movies_table
```
- -sql : 用數字序列的方式命名 migration 檔案 (如0001, 0002, ...)
  預設是用 unix timestamp
- -ext : 指定 migration 檔案的 extension 名稱
- -dir : 表示我們要把 migration 檔案建立在哪個路徑 (若路徑不存在的話會自動建立)
- _create_movies_table_ 是想要建立的檔案名稱

接著系統會建立兩份 migration 檔案
`000001_create_movies_table.down.sql`
`000001_create_movies_table.up.sql`

之後便可在這兩份檔案寫入對應的 SQL 語法

## 執行 migrate (up)

```shell
$ export GREENLIGHT_DB_DSN='postgres://greenlight:@localhost/greenlight?sslmode=disable'
$ migrate -path=./migrations -database=$GREENLIGHT_DB_DSN up 

# 如果要特定 up 1個版本 
$ migrate -path=./migrations -database=$GREENLIGHT_DB_DSN up 1
```

系統會額外建立 `schema_migrations` 的 table，用來追蹤 migrate 的情況

version|dirty
---|---
2|f

- version : 現在是 migrate 執行到第幾版
- dirty : 如果是 false，代表 migrate 執行過程中沒發生問題

## schema_migrations (goto)
```shell
$ migrate -path=./migrations -database=$EXAMPLE_DSN goto 1
```

直接指定 migrate 到版本1, 系統會 up/down 到對應版本

## 執行 migrate (down)

```shell
$ export GREENLIGHT_DB_DSN='postgres://greenlight:@localhost/greenlight?sslmode=disable'
# 直接用 down 會全部回溯，即 table 會全部被刪掉
$ migrate -path=./migrations -database=$GREENLIGHT_DB_DSN down

# 如果要特定 down 1個版本 
$ migrate -path=./migrations -database=$GREENLIGHT_DB_DSN down 1
```

## 碰對 migrate 發生錯誤

你會看到 `Dirty database version {X}. Fix and force version.`

那就要先修正有問題的 migrate 檔案內的 SQL 語法

接著強致指定 version 要更新到哪個版本

```shell
$ migrate -path=./migrations -database=$GREENLIGHT_DB_DSN force 1000
```

強致指定到 version 1000 

這只會更新 schema_migrations 表格內容

不會跑任何 migration
