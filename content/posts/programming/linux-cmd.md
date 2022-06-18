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
# n flag: do not output the trailing newline
echo -n hello
```

## ls
long list

列出檔案 / 文件夾
```shell
# -l 可以多顯示 detail 資訊
# -a 可以顯示隱藏檔案
# -t 照建立時間排序
# -tr 照建立時間反排序

ls
```

## cd
變更當前目錄位置
```shell
cd <dir>
# 只輸入 cd 不帶參數的話，會回家目錄
cd
# ~
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
# 可以一次建立多個資料夾
mkdir Asia Europe Africa America
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

## man
查看操作手冊

```shell
# 查看 echo 的操作手冊
man echo
```

## &
讓指令可以在 background 執行

```shell
sh my_script.sh &
```

## jq
可以更好識別 json 的印出指令

```shell
# 如果要全部顯示就用 .
# 如果是要搜尋特定節點，比如說 "script"，就用 .scripts
jq . employee.json
```

## sed
stream editor，即指串流編輯

```shell
# 把 8080 字串改成 9090
# s 是指取代
# g 是指 global，或是換成數字，就會變成只改第幾個匹配的項目
# i flag 是指修改檔案
sed -i 's/8080/9090/g'
```

## nohup
當 linux 使用者登出時，不要關閉 app

```shell
# 在背景執行 python app.py，且不會受到使用者登出影響
nohup python app.py &
```

## wc
word count

計算資料的字數/行數

```shell
# 印出 filename 的資訊
wc [filename]
# 印出 換行數 (new line) / 字數 (word) / 位元組數 (byte)

# l flag 是指只要行數
wc -l [file]
```

## eval
可以把環境變數當成指令來執行

[reference](https://linuxhint.com/bash_eval_command/)

```shell
# 把指令做成參數
mycommand="wc -l department.txt"
# 透過 eval 呼叫
eval $mycommand
```

## unset
刪除環境變數

```shell
# 刪除 env 中的 GONORPOXY
unset GONOPROXY
```

## uptime
詢問系統啟動了多久

[man page](https://man7.org/linux/man-pages/man1/uptime.1.html)

```shell
uptime
# result
21:49  up 10:27, 5 users, load averages: 3.69 3.24 4.32
# 21:49    是系統當前時間
# up 10:27 是指已執行時間
# 5 users  是指使用者總連線數
# load average 是指最近 1, 5, 15 中的系統平均負載
```

## type
可以用 type 來看這個指令是什麼類型的

```shell
# 查看 echo 的型別
type echo
# result
echo is a shell builtin
```

## pushd & popd
pushd 會進入對應資料夾，並 push 到 stack

popd 會退回上一次的路徑，並把 stack pop 出來

```shell
pushd /etc
popd
```

## more
進入能滾動的文字顯示畫面

```shell
more new_file.txt
# [space] 往下滾一面
# [enter] 往下滾一行
# [b] 可以回滾一面
# [/] 可以搜尋
# [q] 結束
```

## less
和 more 類似

```shell
less new_file.txt
```

## whatis
用一行敘述描述這個指令是做什麼的

```shell
whatis date
```

## apropos
查詢 keyword 相關功能的命令

```shell
# 查看 mkdir
apropos mkdir
```

## chsh
更換 shell

```shell
chsh
# 系統會要求輸入使用者密碼以及要使用的shell

# 或是直接一道指令
# 例: 把 shell 換成 Bourne Shell
sudo chsh -s /bin/sh shou
```

## env
查看所有環境變數

```shell
env
```

## which
查看某個指令的位置

```shell
# 查看 kubectl
which kubectl
# result
/usr/local/bin/kubectl
```

## ln
建立 Symbolic link

```shell
# 格式 ln -s [目標對象的路徑] [要建立的位置]
# 例: 建立 ~/my_movices 的 symbolic link 指向 /mnt/my_drive/movies
ln -s /mnt/my_drive/movies ~/my_movies
```



