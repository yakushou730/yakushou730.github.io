---
title: "tools"
authors: [yakushou730]
date: 2022-01-10T10:29:47+08:00
description: "tools"
tags: ["programming"]
draft: false
---

## cloc

[https://github.com/AlDanial/cloc](https://github.com/AlDanial/cloc)

計算 檔案數量 / 行數 等等的工具 

可以使用 docker container 來跑，就不用安裝 binary

```shell
# 最後的 . 是指當前目錄，可以換成對應的目錄
docker run --rm -v $PWD:/tmp aldanial/cloc .
```

## commitizen
[https://github.com/commitizen/cz-cli](https://github.com/commitizen/cz-cli#making-your-repo-commitizen-friendly)

可以簡單產生 commitizen 的 git commit

[https://www.conventionalcommits.org/en/v1.0.0/](https://www.conventionalcommits.org/en/v1.0.0/)

安裝:
```shell
npm install commitizen
commitizen init cz-conventional-changelog --save-dev --save-exact
```
