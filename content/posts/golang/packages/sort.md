---
title: "sort"
authors: [yakushou730]
date: 2022-02-25T00:31:11+08:00
description: "sort"
tags: ["golang"]
draft: false
---

## Ints
對 int slice 做排序
```go
sort.Ints(arr)
```

## IntsAreSorted
回傳 true / false 判斷 int slice 是否已經排序
```go
func IntsAreSorted(arr []int) bool
```

## Float64s
對 float slice 做排序
```go
func Float64s(a []float64)
```

## Strings
對 string slice 做排序
```go
func Strings(a []string)
```

## SearchInts
對 `已經排序好的 int arry` 做搜尋

回傳 index

```go
func SearchInts(a []int, n int) int
```

## SearchFloat64s
```go
func SearchFloat64s(a []float64, x float64) int // search for float64 
```

## SearchStrings
```go
func SearchStrings(a []string, x string) int // search for strings
```
