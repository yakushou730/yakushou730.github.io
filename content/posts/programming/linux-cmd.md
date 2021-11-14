---
title: "Linux Cmd"
authors: [yakushou730]
date: 2021-11-14T10:55:14+08:00
description: "Linux Cmd"
tags: ["programming"]
draft: false
---

## find

```shell
# 在 當前資料夾 下對 檔案名稱 *_spec.rb 做搜尋 
$ find . -name "*_spec.rb"
```

## export

```shell
# 環境變數 設定資料庫連線 DSN
$ export GREENLIGHT_DB_DSN='postgres://greenlight:@localhost/greenlight?sslmode=disable'

# 列出環境變數
$ export -p
```
