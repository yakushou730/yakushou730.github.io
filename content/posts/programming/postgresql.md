---
title: "PostgreSQL"
authors: [yakushou730]
date: 2021-11-14T10:42:24+08:00
description: "PostgreSQL"
tags: ["programming"]
draft: false
---

## 介紹

#### 登入機制

peer authentication: 登入過一次後，就不用再驗證，可以直接輸入 `psql` 進入 `cli` 模式

password-based authentication: 要輸入密碼才可以登入的驗證

## 指令

#### Meta command

在 PostgreSQL 的指令，以 \ 為 meta command，
以提供一些有用的操作

```sql
-- see the full list of available meta commands
\?

--  list all databases
\l

-- list tables
\dt

-- list users
\du
```

#### 安裝 extension

安裝 `citext` 的擴充功能

```sql
CREATE EXTENSION IF NOT EXISTS citext;
```

#### 建立使用者

建立名為 `greenlight` 且密碼為 `pa55word` 的帳號

```sql
CREATE ROLE greenlight WITH LOGIN PASSWORD 'pa55word';
```

#### 查看使用者

```sql
\du
```

#### 使用者登入

登入 greenlight 角色 需要登入密碼的話會跳出輸入提示

```shell
psql --host=localhost --dbname=greenlight --username=greenlight
```

#### 刪除使用者

在 SHELL 直接輸入指令

```shell
$ dropuser greenlight
```

#### 顯示當前使用者

```sql
SELECT current_user;
```

#### 建立 database

範例: 建立名為 greenlight 的 database

```sql
CREATE DATABASE greenlight;
```

#### 連接 database

範例: 連接名為 greenlight 的 database

```sql
\c greenlight;
```

#### 顯示 config_file

```shell
$ psql -c "SHOW config_file;" 
               config_file               
-----------------------------------------
 /usr/local/var/postgres/postgresql.conf
(1 row)
```

#### 查看 table schema

查看 movies table schema

```sql
\d movies
```

## Troubleshooting
1. 當 postgresql server 開起不了的時候可以下指令看 log 

```shell
$ tail -n 10 /usr/local/var/log/postgres.log   
2021-11-14 10:40:37.439 CST [8326] FATAL:  database files are incompatible with server
2021-11-14 10:40:37.439 CST [8326] DETAIL:  The data directory was initialized by PostgreSQL version 12, which is not compatible with this version 14.1.
```

2 如何移除 postgresql 並重新透過 brew 安裝

[**How to completely uninstall and reinstall Homebrew Postgres**](https://blog.testdouble.com/posts/2021-01-28-how-to-completely-uninstall-homebrew-postgres/#completely-uninstalling-a-homebrew-installation-of-postgres)
