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

欄位 `net.ipv4.ip+forward = 1`


![devops/devops-route-example.png](/devops/devops-route-example.png)
