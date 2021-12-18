---
title: "mapping and analysis"
authors: [yakushou730]
date: 2021-12-17T01:19:57+08:00
description: "mapping and analysis"
tags: ["udemy", "elasticsearch"]
draft: false
---

## Introduction to analysis
- `_source` object 是做 index document 但卻不是用來作為搜尋的內容欄位
- 分析字串:
  - analyzer
    - character filter
    - tokenizer
    - token filter
    - 舉例 (filter 和 tokenizer 種類繁多，僅舉例簡單情境):
  - 分析 `I REALLY like beer!`
    - character filter 做成 `I REALLY Like beer!`
    - Tokenizer 做成 `["I", "REALLY", "Like", "beer"]`
    - Token Filter 做成 `["i", "really", "like", "beer"]`
- 儲存的字串會被切成較小的字串儲存 (Tokenizer)

## Using the Analyze API

```
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "analyzer": "standard"
}

# result
{
  "tokens" : [
    {
      "token" : "2",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<NUM>",
      "position" : 0
    },
    {
      "token" : "guys",
      "start_offset" : 2,
      "end_offset" : 6,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "walk",
      "start_offset" : 7,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "into",
      "start_offset" : 12,
      "end_offset" : 16,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "a",
      "start_offset" : 19,
      "end_offset" : 20,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "bar",
      "start_offset" : 21,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "but",
      "start_offset" : 26,
      "end_offset" : 29,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "the",
      "start_offset" : 30,
      "end_offset" : 33,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "third",
      "start_offset" : 34,
      "end_offset" : 39,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "ducks",
      "start_offset" : 43,
      "end_offset" : 48,
      "type" : "<ALPHANUM>",
      "position" : 9
    }
  ]
}
```

下面的範例和上面的範例會有一樣結果

因為 analyzer 是 char_filter + tokenizer + filter 組成的

而下面的範例組成為 standard analyzer
```
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}
```
