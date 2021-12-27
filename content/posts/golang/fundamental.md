---
title: "fundamental"
authors: [yakushou730]
date: 2021-12-27T16:10:27+08:00
description: "fundamental"
tags: ["programming","golang"]
draft: false
---

## 如何在 go get 的時候可以拉到 private repo
概念:

因為是 private repo 的關係， 所以要讓 go 有權限去拉

拉的時候是透過 git，所以要設定具有權限的 token 給 git

1. 在個人的 github settings 建立一個可以拉取 private repo 的 token
2. 把 token 記下以後，在本機操作 git cli
3. 用 copy 下來的 token 取代下列的 GITHUB_TOKEN

```shell
$ git config --global url."https://${GITHUB_TOKEN}@github.com".insteadOf "https://github.com"
```
