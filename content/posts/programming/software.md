---
title: "software"
authors: [yakushou730]
date: 2021-12-27T18:04:23+08:00
description: "software"
tags: ["programming"]
draft: false
---

## Declarative vs imperative language
Declarative programming (宣告式)
- **What**
- 代表為 functional programming
- (function 之間不會互相共用狀態)
- 程式碼通常是可以直接看到在做什麼 (重視 what)
 
Imperative programming (命令式)
- **how to do**
- 代表為 OOP 
- (狀態互相依賴)
- 看得見邏輯運算過程 (how to do)

## Repository Pattern
Repository Pattern 是在 domain 和 data mapping 的中間層

透過 collection-like 的 interface 來存取 domain object

換句話說可以是 application 和 data access logic 的中間層

好處:
- 修改邏輯的時候只要改一個地方
- 測試 controller 變簡單了，不用真實存取資料庫資料

舉例:
EmployeeController -> Employee Repository -> SQL server Database

EmployeeController 提供 API action

Employee Repository 提供存取資料的 interface，如 GetAll(), GetByID(), Insert(), Update(), Delete()

然後對 Repository 的介面另外實作如何存取 data (可以區分 mysql, postgres...)

實作上 domain model 不需要知道 data store 的資訊或 tag，就是個單純有商業邏輯的 entity

會另外開一個 data transfer object，用來做資料庫對應 entity 的實作，不需要商業邏輯

範例:

使用 repository pattern 取得 movie 資料
1. 有一個 movie model，只用來作為 entity 的資料
2. 有一個 movieRepository 的 interface，裡面定義了操作資料庫的 method
3. 實作 mysqlRepository 或 postgresRepository 實例，滿足 movieRepository interface
4. server 啟動的時候，設定操作資料庫的 movieRepository 是哪種實例
5. handler 內可以對對應的實例可以呼叫 method 來取得資料，並傳回 domain entity
6. 在 handler 內把傳回的 domain entity 包成 JSON response 傳出




