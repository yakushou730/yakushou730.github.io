---
title: "strings"
authors: [yakushou730]
date: 2022-02-19T17:44:28+08:00
description: "strings"
tags: ["golang"]
draft: false
---
## HasPrefix
檢驗字串的 prefix

`strings.HasPrefix(s, prefix string) bool`

## HasSuffix
檢驗字串的 suffix

`strings.HasSuffix(s, suffix string) bool`

## Contains
檢驗字串是否包含 substring

`strings.Contains(s, substr string) bool`

## Index
指出第一個符合的 substring 是在哪個位置，沒找到的話回傳 -1

`strings.Index(s, str string) int`

## LastIndex
指出最後一個符合的 substring 是在哪個位置，沒找到的話回傳 -1

`strings.LastIndex(s, str string) int`

> 若是非 ACSII 的話，用 `strings.IndexRune(s string, ch int) int`

## IndexRune
非 ASCII 找 index 時用的

`strings.IndexRune(s string, ch int) int`

## Replace
取代字串，從頭開始 n 次，全部取代的話用 `-1`

`strings.Replace(str, old, new string, n int)`

##  Count
計算 substring 不重疊出現次數

`strings.Count(s, str string) int`

## Repeat
重複 count 數量的字串

`strings.Repeat(s string, count int) string`

## ToLower
回傳原本字串都轉成小寫的 copy

`strings.ToLower(s) string`

## ToUpper
回傳原本字串都轉成大寫的 copy

`strings.ToUpper(s) string`

## TrimSpace
移除全部的空白

`strings.TrimSpace(s)`

## Trim
移除全部特定的字串

`trings.Trim(s, str)`

## TrimLeft

## TrimRight

## Fields
用 空白 或 連續空白 來分割字串

回傳 []string

`strings.Fields(s)`

## Split
用特殊分割符號來切字串

`strings.Split(s, sep)`

## Join
把 []string  用特殊字符串起來

`strings.Join(sl []string, sep string)`

## Read
`strings.NewReader(str)`

先產生一個指標指向 reader value

用 Read() 可以把資料讀取為 `[]byte`

## ReadByte
同上先呼叫 `strings.NewReader(str)` 後

用 ReadByte() 讀下一個 byte

## ReadRune
同上先呼叫 `strings.NewReader(str)` 後

用 ReadRune() 讀下一個 rune

