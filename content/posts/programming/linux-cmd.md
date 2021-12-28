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

# 如果有 query params 的話，整段 url 要用雙引號包起來
$ curl "localhost:4000/v1/movies?page_size=2&page=2"
```

## for

使用 for loop

```shell
# 使用 loop 6次，每次都打 curl
$ for i in {1..6}; do curl http://localhost:4000/v1/healthcheck; done
```

## pgrep

查看正在運行的程序，並列出 PIDs (Process IDs)

[the man page](https://linux.die.net/man/1/pgrep)

```shell
# -l 表示查詢結果多顯示程序名稱，這邊的 api 是要查詢的程序名稱
$ pgrep -l api
```

## pkill

送出特別的訊號給程序 (預設是SIGTERM)

[the man page](https://linux.die.net/man/1/pkill)

```shell
# 對 api 的程序送出 SIGKILL 訊號
$ pkill -SIGKILL api
```

## run multiple commands in one line

指令是可以一次下複數個的

```shell
# 下完 curl 以後馬上關閉名為 api 的 process
$ curl -d "$BODY" localhost:4000/v1/users & pkill -SIGTERM api &
```

## w

查看目前登入系統的使用者有哪些人

以及他們正在使用的程式

## read

read 可以提示使用者輸入字串

並賦值給變數

```shell
# 將使用者輸入的字串賦值給 PASSWORD
$ read -p "Enter password for root DB user" PASSWORD
```

## ssh

透過 ssh 連線到遠端機器

```shell
# 用 ubuntu 帳號連線至 172.1.1.1
$ ssh ubuntu@172.1.1.1
```

ip 和 username 是可以先寫在 ~/.ssh/config 檔案內的

範例:
```shell
Host shou_prod
HostName 172.1.1.1
User ubuntu
```

這樣的話可以用以下指令連線到機器

```shell
$ ssh shou_prod
```

## ufw

uncomplicated firewall

可以用簡單的指令來操作防火牆

範例

```shell
# 允許 port 22
$ ufw allow 22
# 允許 port 80
$ ufw allow 80/tcp
# 允許 port 443
$ ufw allow 443/tcp
# 封鎖 port 4000
$ ufw deny 4000
```

[man page](http://manpages.ubuntu.com/manpages/bionic/man8/ufw.8.html)
