---
title: "Mysql"
authors: [yakushou730]
date: 2022-01-27T15:05:19+08:00
description: "mysql"
tags: ["programming","sql"]
draft: false
---

## 使用者相關
要注意的是 User, Host 的搭配會影響到使用者可不可以登入

或是登入後的權限不同

`%` 指的是 `localhost`

`::1` 指的是 Ipv6 的 `localhost`

[How to delete or remove a MySQL/MariaDB user account on Linux/Unix](https://www.cyberciti.biz/faq/how-to-delete-remove-user-account-in-mysql-mariadb/)

[How To Create a New User and Grant Permissions in MySQL](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql)

`Grant_Priv` 欄位是指可不可以更換權限

```mysql
# 檢視 user 欄位
desc mysql.user;
# 顯示當前使用者
SELECT user();
# 搜尋使用者
SELECT * FROM mysql.user;
# 建立使用者
CREATE USER 'shou'@'%' IDENTIFIED BY 'password';
CREATE USER 'academy'@'172.123.0.0/255.255.0.0' IDENTIFIED BY 'password';
# 更改使用者密碼
ALTER USER 'shou'@'%' IDENTIFIED BY 'newPass';
# 刷新權限
FLUSH PRIVILEGES;
# 給予使用者權限
GRANT ALL PRIVILEGES ON *.* TO 'shou'@'%';
GRANT SELECT ON *.* TO 'shou'@'%';
# 查詢權限
SHOW GRANTS FOR 'shou'@'%';
# 刪除使用者
DROP USER 'shou'@'%';
```

## Reset table index
```mysql
ALTER TABLE [table_name] AUTO_INCREMENT = 0;
```
