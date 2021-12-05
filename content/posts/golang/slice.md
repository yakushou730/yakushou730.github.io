---
title: "Slice"
authors: [yakushou730]
date: 2021-11-16T22:59:16+08:00
description: "Slice"
tags: ["programming","golang"]
draft: false
---

## 建立空的 Slice

```go
var movies []*Movie
movie = &Movie{}
// 這樣子建立出來的 movies slice 是 nil
// 但已經足夠用來做 append
// 不需要寫成 movies := []*Movie{}
movies = append(movies, movie)
```

## 把 array 轉成 slice

```go
// hash 是 [32]byte 的 byte array
hash := sha256.Sum256(info)
// b.Hash 是 byte slice
b.Hash = hash[:]
```
