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

