---
title: "strconv"
authors: [yakushou730]
date: 2022-02-19T17:44:34+08:00
description: "strconv"
tags: ["golang"]
draft: false
---

## IntSize
回傳 int 的 size 大小

## Itoa
int 轉 string

`strconv.Itoa(i int) string`

## FormatFloat
把 float 轉成 string

`strconv.FormatFloat(f float64, fmt byte, prec int, bitSize int) string `

## Atoi
字串轉 int

`strconv.Atoi(s string) (i int, err error)`

## ParseFloat
字串轉 float

`strconv.ParseFloat(s string, bitSize int) (f float64, err error)`
