---
title: "The Linux Basics Course"
authors: [yakushou730]
date: 2022-01-24T00:16:26+08:00
description: "The Linux Basics Course"
tags: ["programming","devops"]
draft: false
---

[The Linux Basics Course](https://github.com/kodekloudhub/linux-basics-course)

## Working with Shell - I
`/` 是根目錄

`/home` 是所有使用者的家目錄放置的位置

比如一個使用者叫做 shou，那 shou 的家目錄就在 `/home/shou`

一般使用者不能訪問其他使用者的家目錄

家目錄的標記為 `~` 英文是 tilde

flag 通常是 一個單字 配一個 `-` hyphen

- `.` 代表當前目錄
- `..` 代表上一層目錄

通常指令要找 help 的話可以在指令後方加上 `-h` 或 `--help`

`$HOME` 是家目錄的環境變數

**Shell Types**
- Bourne Shell (sh)
- C Shell (csh or tcsh)
- Korn Shell (ksh)
- Z Shell (zsh)
- Bourne again Shell (bash)

```shell
# 查看當前是用哪種 shell
echo $SHELL
```

Bash Shell Features
- auto completion
  - 按 tab
- alias
  - `alias dt=date`
- history
  - `history`

> $LOGNAME 是登入名稱

shell 透過 $PATH 去找所有系統可以執行的指令位於的路徑

> 串接 PATH
> 
> export PATH=$PATH:[要串接的路徑]
> 
> EX: export PATH=$PATH:/opt/obs/bin

**Bash Prompt**

`[~]$` 中的 `$` 是 bash prompt symbol

```shell
# 查看 bash prompt 的格式
echo $PS1
# 可客製化
PS1="[\d \t \u@@\h:\w ] $ "
# result
[Thu Mar 12 22:12:54 bob@caleston:~ ] $
```

[Bash Prompt Tutorial](https://linuxconfig.org/bash-prompt-basics)

```shell
# 把字串插入 /home/bob/.profile 的最下面
echo 'export PROJECT=MERCURY' >> /home/bob/.profile
```
