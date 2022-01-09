---
title: "basics"
authors: [yakushou730]
date: 2022-01-02T09:27:51+08:00
description: "basics"
tags: ["programming","devops"]
draft: false
---

## Environment variable
```
SHELL: 系統使用了哪種 shell
```

## 確認 OS Version
```shell
# 印出所有 release 版本
# * 是指 wildcard
ls /etc/*release*
# 印出當前版本細節
cat /etc/*release*
```

## package managers
**RPM (Red Hat Package Manager)**
- RPM 不管 dependency
  - CentOS
  - Red Hat Enterprise Linux
  - Fedora

package 會被包裝成 <檔名>.rpm

```shell
# install package
rpm -i telnet.rpm
# uninstall package
rpm -e telnet.rpm
# query package
rpm -q telnet.rpm
# query all installed package
rpm -qa
```

**YUM**
包裝過 rpm，會順便安裝 dependency
- 底層是 rpm
- 透過 `/etc/yum.repos.d` 決定 yum 要去哪邊載 package

```shell
# install package
yum install ansible
# -y flag 代表預設都同意
yum install -y ansible
# 列出 repo list
yum repolist
# 列出 repositories 在哪裡被 configured
ls /etc/yum.repos.d
# 印出對應的 repo 路徑 (以 CentOS-Base.repo 為例)
cat /etc/yum.repos.d/CentOS-Base.repo
# 印出安裝的 package
yum list
# 搜尋安裝的 package
yum list <package name>
# 移除 package
yum remove <package name>
# -y flag 代表預設都同意
yum remove -y <package name>
# - 可以指定版本
yum install ansible-2.8.11
```

## services
安裝後的軟體或程序，在執行的時候 (可能在背景) 稱為 service

```shell
# 可能會需要提供 sudo 權限來操作
# start httpd service
service httpd start
# or
systemctl start httpd

# stop httpd service
systemctl stop httpd
# check httpd service status
systemctl status httpd
# configure httpd to start at startup
systemctl enable httpd
# configure httpd to not start at startup
systemctl disable httpd
```

範例:
```shell
# 原本的呼叫方式
/usr/bin/python3 /opt/code/my_app.py
# 希望調整成能夠透過 systemctl 簡易呼叫或停止
# 或是可以再開機時被呼叫
# 需要透過 systemd
```
systemctl 是用來控管 systemd service 的指令

systemd service 透過 systemd unit file 控管

路徑可能為 `/etc/systemd/system/`

my_app.service
```
[Unit]
Description=My python web application

[Service]
ExecStart=/usr/bin/python3 /opt/code/my_app.py
ExecStartPre=/opt/code/configure_db.sh
ExecStartPost=/opt/code/email_status.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

`ExecStartPre` 是指說在跑 service 之前要執行什麼
`ExecStartPost` 是指說在跑 service 之後要執行什麼

`[Install]` 區段會讓電腦開機的時候自動開啟

```shell
# 重新載入 systemd configuration
systemctl daemon-reload
# 接下來即可透過 systemctl 正常呼叫 service
systemctl start my_app
# 查看組態設定
systemctl cat my_app
```

## Virtual Box networking
當裝置的網路介面連接到網路的時候，都會獲取一個 ip
```shell
# 顯示 ip 資訊
ip addr show
```

Network adapter
- Host Only Network
  - 以電腦為 host，把所有的 virtual machine 網路都接起來
  - 例: 1台 電腦內開了 4個 virtual machine
    - 透過 Host-Only network 192.168.5.0
    - 電腦 host ip 為 192.168.5.1
    - virtual machine 1 -> 192.168.5.2
    - virtual machine 2 -> 192.168.5.3
    - virtual machine 3 -> 192.168.5.4
    - virtual machine 4 -> 192.168.5.5
    - 電腦和 virtual machine 可互通
    - 無法對外連線
    - 電腦對外的 ip 為 192.168.1.10
    - 電腦外部 LAN 的 ip 為 192.168.1.0
- NAT network (Network Address Translation)
  - 讓 host 內的 virtual machine 可以對外連線
  - virtual machine 彼此間也可以溝通
  - virtual machine 192.168.5.2 送出 request 給 192.168.1.11 時
    - 先把 FROM IP 從 192.168.5.2 轉成 192.168.1.10 後送出
    - 192.168.1.11 資料庫收到 request 後回應 response
    - NAT 收到 response 後再把 FROM IP 從 192.168.1.10 轉回 192.168.5.2
    - 外部服務不會注意到是 192.168.1.10 的內部 virtual machine，即從外部無法存取內部資訊
- NAT
  - host 內的 virtual machine 可以對外連線
  - 但無法對內部的 virtual machine 彼此連線
- Bridge Network
  - 視為外部網段 LAN 對內的 extension 
  - 內部 virtual machine 不會有自己的網段，直接使用外部網段 192.168.1.0
    - virtual machine 1 -> 192.168.1.12
    - virtual machine 2 -> 192.168.1.13
    - virtual machine 3 -> 192.168.1.14
    - virtual machine 4 -> 192.168.1.15

Port Forwarding
- 就是在 host 開一個 port，可以把進來的 request 對應到 內部的 virtual machine 的 port

## Vagrant
```shell
# create vagrant file with os
vagrant init centos/7
# start a virtual machine
vagrant up
# 連線進 vagrant virtual machine
vagrant ssh
# shutdown
vagrant halt
# 查看狀態
vagrant status
# 重新載入 vagrant file for virtual machine
vagrant reload
```

## Network basics
**Switching**

Switch 是用來讓 多台裝置彼此之間可以溝通連線的裝置

假設有 A B 兩台裝置，中間透過一台 switch 設定即可溝通
```shell
# 取得 裝置的 網路介面 (拿到 eth0)
# 列出清單 和 調整 host 上的 interface
ip link 
# 查看 IP address assign 給 interface 的對應
ip addr 
# 假設 switch 網段為 192.168.1.0
# A 裝置設定
ip addr add 192.168.1.10/24 dev eth0
# B 裝置設定
ip addr add 192.168.1.11/24 dev eth0
```

> 要注意 ip addr add 的指令如果重開機的話就失效了
> 
> 要保留的話要寫到 etc network interface file

**Routing**

Router 是讓兩個不同 switch 網路的裝置可以互相溝通

假設有兩組 switch，一組為 192.168.1.0 (裝置A,B)，一組為 192.168.2.0 (裝置C,D)

則在 Router 的端口 ip 第一組設定為 192.168.1.1，另一組為 192.168.2.1 

**Gateway**

> 如果 network 是房間的話，gateway 就是對外的門
> 
> 系統需要知道那個門是去哪的

```shell
# 顯示 routing table
route
```

![devops/devops-gateway-example.png](/devops/devops-gateway-example.png)

假設 B 192.168.1.11 要連去 C 192.168.2.10
```shell
# 在 B 那台下指令
ip route add 192.168.2.0/24 via 192.168.1.1
# C 那台也要下指令
ip route add 192.168.1.0/24 via 192.168.2.1
# 假設 C 那台要連去網際網路 172.217.194.0 的話
ip route add 172.217.194.0/24 via 192.168.2.1
# 假設 C 那台要連去任何網際網路的話
# default 和 0.0.0.0 一樣意思
ip route add default via 192.168.2.1

# 檢查 ip forwarding 是否在 host 上啟用
cat /proc/sys/net/ipv4/ip_forward
# host 是否可以在 interface 之間做 forward 溝通，設定在 /proc/sys/net/ipv4/ip_forward
# default 是 0，代表 no forward
cat /proc/sys/net/ipv4/ip_forward
# 可以自己改數字
echo 1 > /proc/sys/net/ipv4/ip_forward
```

因為重啟後 ip_forward 設定就沒了，所以要寫到對應的 file 設定

檔名 `/etc/sysctl.conf`

欄位 `net.ipv4.ip_forward = 1`

![devops/devops-route-example.png](/devops/devops-route-example.png)

## DNS
本機的 IP 對 host 的 mapping 表放在 `/etc/hosts` 內

透過設定 `/etc/hosts` 的設定方式稱為 Name Resolution

但每台機器都要設定的話，很不彈性，萬一某個 ip 換了很麻煩

所以把這個對應表拉出去獨立的機器上 就稱為 DNS

每台機器都知道 DNS 在哪邊，設定在 `/etc/resolv.conf` 

優先權 `/etc/hosts` > `/etc/resolv.conf`

但可以在 `/etc/nsswitch.conf` 設定優先順序

`/etc/resolv.conf` 上面的 nameserver 不一定涵蓋全部的 domain

所以可以自己加上 nameserver (認得出這個 domain 的)

==

`www.google.com` 拆分
- `.` 是 root
- `com` 是 top level domain
- `google` 是 domain name
- `www` 是 subdomain，其他如 `mail`, `drive`, `maps`, `apps` ... 等等

透過 search 方式 mapping 名稱
```shell
# dns
192.168.1.10 web.mycompany.com
# 如果不想要用這麼長的名字連線
# 只想用 web 字串的話
# 要在 /etc/resolv.confg 另外設定
search mycompany.com
# 這樣子ping web 等同於 ping web.mycompany.com
ping web
ping web.mycompany.com
```

**Record Types**
- `A`: ip to host
- `AAAA`: ipv6 to host
- `CNAME`: one name to another name

**nslookup**
用來測試 dns 解析

要注意的是這會忽略本機的 hosts 設定

```shell
nslookup www.google.com
```

**dig**
也是用來測試 dns 解析的

```shell
dig www.google.com
```

## Application Basics
Language:
- compiled:
  - Java, C, C++, ...
  - `Develop Source Code` -> `Compile` -> `Run`
- interpreted:
  - Python, node, Ruby, ...
  - `Develop Source Code` -> `Run`

## IPs and Ports
假設今天有一台電腦 開了兩個 IP 入口對外

若要開放不指定 IP 的話，就用 `0.0.0.0`

網路介面 `l0` 是指 `127.0.0.1`

## SSL & TLS Basics
非對稱金耀 asymmetric encryption
- Private Key
- Public Lock (或可稱 public key)

以 SSH 為例
- 在遠端機器上放入 public lock
  - 只有有 private key 的人可以連線進入
  - 放在遠端機器的 ~/.ssh/authorized_keys

以 HTTPS 為例
- 遠端機器自己先建立好一組金鑰對 (asymmetric的)
  - 假設 `my-bank.key mybank.pem`
  - 使用者只需要 對稱金鑰
  - 遠端機器 (server) 有 非對稱金鑰 
  - CA 使用非對稱金鑰
  - CA 的 public key 在每個 browder 都有
  1. 遠端機器請求 CA 幫忙驗證自己是合法的 (送出 CSR)
  2. CA 確認後用 `CA 的 private key` 簽名
  3. 遠端機器拿到簽名後的 certificate後，設定在機器上 (configuration)
  4. 當使用者要存取遠端機器的網頁時，遠端機器送出 certificate 和 遠端機器的 public lock
  5. 使用者的瀏覽器用 `CA public key` 解開簽證，確認 `遠端機器的 public lock` 是合法的
  6. 之後使用者用 `遠端機器的 public lock` 鎖住 `使用者的 key` 傳給遠端機器
  7. 如此只有遠端機器可以用 `遠端機器的 private key` 解開 `遠端機器的 public lock` 拿到 `使用者的 key`
  8. 之後 遠端機器就用 `使用者的 key` 鎖住要給使用者的資料傳出去
  9. 使用者可以用 `自己的 key` 解密得到資料
  10. 使用者都用 `自己的 key` 加密後傳給 遠端機器
  11. 遠端機器用 `使用者的 key` 解密

> 這樣在溝通中，使用者的 key 都不會裸露出來

現在的瀏覽器都內建了 驗證 certificate 的機制 (Certificate Authority 組織會把他們的 public key 安裝到瀏覽器上)

可以知道連線安不安全

遠端機器如何確認 client 是同一個?
1. 提出 client 要提供 certificate
2. 然後 client 就要生成 非對稱金鑰 且也要對 CA 提出 CSR 並拿到 CERT
3. client 把自己的金鑰和 CERT 傳給 遠端機器 

> PKI (Public Key Infrastructure)

非對稱金鑰只能一把加密 另一把解密

同一把鑰匙不能同時做到加解密

> Convention
> 
> Certificate (Public Key) 通常用 *.crt 或 *.pem
> 
> Private Key 通常用 *.key 或 *.key.pem 

## JSON path
JSON path 是對 JSON 的 query 語法 (dictionary)

就像 SQL 是對資料庫一樣

最外層的 `{}` 是指 root element 用 `$` 代表

而 JSON path query 的結果都是在 `[]` 內，用中括號包起來

```json
{
  "car": {
    "color": "blue",
    "price": "$20000"
  },
  "bus": {
    "color": "white",
    "price": "$120000"
  }
}
```

**Query**

- Get car details: `$.car`
```json
[
  {
    "color": "blue",
    "price": "$20000"
  }
]
```

- Get bus details: `$.bus`
```json
[
  {
    "color": "white",
    "price": "$120000"
  }
]
```
- Get car's color: `$.car.color`
```json
[
  "blue"
]
```

另一個範例 for 陣列
```json
[
  "car",
  "bus",
  "truck",
  "bike"
]
```

- Get the 1st element
```json
$[0]

// result
[
  "car"
]
```

- Get the 4th element
```json
$[3]

// result
[
  "bike"
]
```

- Get the 1st and 4th element
```json
$[0,3]

// result
[
  "car",
  "bike"
]
```


另一個範例
```json
[
  12,
  43,
  23,
  12,
  56
]
// 要找出大於 40 的
$[ ? ( @ > 40 ) ]

// 結果
[
  43,
  56
]
```

`@` 是指 item in loop

- `@ == 40`
- `@ in [40,43,45]`
- `@ != 40`
- `@ nin [40,43,45]`

複雜的 query 例子
```json
$.car.wheels[?(@.location == "rear-right"].model
```

[JSONPath playground](https://jsonpath.com/)
