---
title: "docker"
authors: [yakushou730]
date: 2022-06-11T23:19:58+08:00
description: "docker"
tags: ["programming","devops"]
draft: false
---

## 清除沒用到的物件

WARNING! This will remove:
- all stopped containers
- all networks not used by at least one container
- all dangling images
- all dangling build cache

```shell
docker system prune
```
