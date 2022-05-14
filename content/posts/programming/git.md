---
title: "git"
authors: [yakushou730]
date: 2022-01-02T09:35:24+08:00
description: "git"
tags: ["programming","git"]
draft: false
---

## 使用標籤 tag
```shell
# 建立標籤
git tag v1.0.0 -am "description for this tag v1.0.0"

# 刪除標籤
git tag -d v1.0.0

# 把標籤推到 github 上 (預設的 git push 是不推 tag 的，所以要自己下 flag 指令)
git push --tags
```
