---
title: "json"
authors: [yakushou730]
date: 2021-11-23T17:08:07+08:00
description: "json"
tags: ["golang"]
draft: false
---

## tag 標記用法

範例 `json:"name,omitempty"`

這個欄位名稱為 name，如果欄位內容是空值的話，則欄位會被忽略

範例 `json:"-"`
不管欄位名稱是什麼，都會忽略不印出
