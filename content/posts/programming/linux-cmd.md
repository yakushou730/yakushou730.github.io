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

# 可以用來下載資料
# -O flag 是指把 curl 回來的資訊輸出成檔案
$ curl http://www.some-site.com/some-file.txt -O
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
# ubuntu    -> username
# 172.1.1.1 -> hostname
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

## echo
印出資訊

```shell
echo Hi
```

## ls
列出檔案 / 文件夾
```shell
ls
```

## cd
變更當前目錄位置
```shell
cd <dir>
```

## pwd
顯示當前目錄位置
```shell
pwd
```

## mkdir
建立資料夾
```shell
mkdir <new directory>
# -p flag 會把不存在的資料夾一並建出來
mkdir -p /tmp/asia/india/bangalore
```

## multiple commands
一次執行多個指令 (用分號隔開)
```shell
cd <directory>; mkdir <new directory>; pwd 
```

## rm
刪除目錄
```shell
# -r flag 是指 recursive，會把目錄底下的東西也全刪了
rm -r /tmp/my_dir
```

## cp
複製目錄
```shell
# -r flag 是指 recursive，會把目錄底下的東西也全複製
cp -r my_dir1 /temp/my_dir1
# -v flag 會印出做了什麼
cp -v empty_file.txt empty_dir/
```

## touch
建立新文件
```shell
touch new_file.txt
```

## cat
對文件操作
```shell
# 對 new_file.txt 插入文字
# 用 ctrl+D 可以跳出輸入畫面
# 其中 > 稱為 redirection symbol
cat > new_file.txt
# 印出文件內容
cat new_file.txt
```

## mv
move (rename) 檔案
```shell
mv new_file.txt sample_file.txt
```

## whoami
印出當前使用者
```shell
whoami
```

## id
取得當前使用者的詳細資料
```shell
id
```

## su
更換使用者 (switch user)

```shell
su <other username>
```

## sudo
superuser do

賦予使用者 root 權限執行指令 (不會切換成 root 使用者)
```shell
sudo <指令>
```

## wget
下載檔案用的
```shell
# -O flag 是指定輸出名稱
wget http://www.some-site.com/some-file.txt -O some-file.txt
```





