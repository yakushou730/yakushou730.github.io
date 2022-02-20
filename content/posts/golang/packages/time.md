---
title: "time"
authors: [yakushou730]
date: 2022-02-19T18:27:51+08:00
description: "time"
tags: ["golang"]
draft: false
---

## 基本用法
```go
t := time.Now()
fmt.Printf("%02d.%02d.%4d\n", t.Day(), t.Month(), t.Year()) // e.g.: 29.10.2019
```
`Duration` 是指兩個時間相減的 nano second (int64)

`Location` 會 mapping 時區

> UTC: Universal Coordinated Time

## Since
回傳過了多久

`Since(t Time)`

## Format
把時間轉成特定格式

`func (t Time) Format(s string) string`

可也以是 `time.ANSIC` 或 `time.RFC822`

```go
t := time.Now().UTC()
fmt.Println(t.Format("02 Jan 2006 15:04"))// e.g: 29 Oct 2019 11:00
```

## Sub
時間相減出來的時間差

`delta := end.Sub(start)`


