---
title: "Linux Cmd"
authors: [yakushou730]
date: 2021-11-14T10:55:14+08:00
description: "Linux Cmd"
tags: ["programming"]
draft: false
---

## find

搜尋檔案

```shell
# 在 當前資料夾 下對 檔案名稱 *_spec.rb 做搜尋 
$ find . -name "*_spec.rb"
```

## export

設定環境變數

```shell
# 環境變數 設定資料庫連線 DSN
$ export GREENLIGHT_DB_DSN='postgres://greenlight:@localhost/greenlight?sslmode=disable'

# 列出環境變數
$ export -p
```

## df

顯示硬碟使用量

short for **disk free**

Filesystem: 檔案系統
Size: 總容量
Used: 已用容量
Avail: 剩餘容量
Use%: 已用百分比
Mounted on: 掛載點

```shell
# -h 以容易識別的格式顯示硬碟空間使用量
$ df -h

# -T 顯示每個分割區所屬的檔案系統名稱
$ df -T
```

## free

顯示記憶體使用情況

```shell
# -m 以 MB 作為單位顯示
$ free -m
```

## curl
可以用來做連線資料傳輸

[the man page](https://curl.se/docs/manpage.html)

```shell
# -d 傳送 data，系統會用 POST
$ BODY='{"title":"The Breakfast Club","year":1986, "runtime":"96 mins","genres":["drama"]}'
$ curl -d "$BODY" localhost:4000/v1/movies

# -i 回傳的時候多顯示 Response header
$ curl -i localhost:4000/v1/healthcheck 

# -X 指定 http method
$ BODY='{"title":"Black Panther","year":2018,"runtime":"134 mins","genres":["sci-fi","action","adventure"]}'
$ curl -X PUT -d "$BODY" localhost:4000/v1/movies/2

# -w 顯示額外資訊
$ curl -w "\nTime: %{time_total}s \n" localhost:4000/v1/movies/2

# 同時執行多個 request
$ curl localhost:4000/v1/movies/1 & curl localhost:4000/v1/movies/1 &
```
