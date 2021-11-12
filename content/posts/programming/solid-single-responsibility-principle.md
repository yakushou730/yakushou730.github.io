---
title: "SOLID - Single Responsibility Principle"
authors: [yakushou730]
date: 2021-11-10T18:24:37+08:00
description: "SOLID - Single Responsibility Principle"
tags: ["programming"]
draft: false
---

SOLID 是軟體開發的準則

分別代表

- **Single Responsibility Principle (SRP - 單一職責原則)**
- **Open-Closed Principle (OCP - 開放封閉原則)**
- **Liskov Substitution Principle (LSP - Liskov替換原則)**
- **Interface Segregation Principle (ISP - 介面隔離原則)**
- **Dependency Inversion Principle (DIP - 依賴反轉原則)**

本篇是簡單整理 SRP 的內容

SRP 的定義是單一職責，類別內只做單一的事情

即如果有要改動這個類別的話，只會有一個原因

不會有多數需求要修改到這個類別

舉例來說 一個 Journey 類別，裡面可以 新增/移除 複數個觀光景點

則這個類別的職責就是只要負責記錄觀光景點就好

假設今天需要一個功能是產生報表

或許直接加方法在這個類別可以簡單的實作

但卻破壞了 SRP 原則，這個時候就應該要把產生報表的行為移到另一個類別來負責

引入的參數可以是這個 Journey 物件

讓這個 Journey 類別只負責記錄觀光景點，不多做不相干的事情

### 參考資料
1. [SOLID 之 單一職責原則（Single responsibility principle）](https://ithelp.ithome.com.tw/articles/10191955)
