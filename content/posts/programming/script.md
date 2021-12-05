---
title: "script"
authors: [yakushou730]
date: 2021-11-29T22:01:31+08:00
description: "sript"
tags: ["programming"]
draft: false
---

範例 script.sh

```shell
$ touch script.sh
$ chmod +x script.sh
```

檔案內容

```shell
#!/bin/zsh

GREENLIGHT_DB_DSN='postgres://greenlight:@localhost/greenlight?sslmode=disable'
echo ${GREENLIGHT_DB_DSN}
```

運行 script.sh

```shell
$ ./script.sh
```


## Reference
[簡明 Linux Shell Script 入門教學](https://blog.techbridge.cc/2019/11/15/linux-shell-script-tutorial/)
