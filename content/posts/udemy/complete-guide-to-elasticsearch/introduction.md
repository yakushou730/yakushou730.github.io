---
title: "introduction"
authors: [yakushou730]
date: 2021-12-14T01:13:54+08:00
description: "introduction"
tags: ["udemy", "elasticsearch"]
draft: false
---

## Introduction to Elasticsearch
- 儲存一筆資料的單位是 document (類似 RDB 的 row)
- Document 的欄位叫做 fields (類似 RDB 的 column)
- 儲存的格式像 json
- client 透過 REST API 和 Elasticsearch 溝通

## Overview of the Elastic Stack
- Kibana
  - 分析 / 視覺畫的平台
  - 透過 Elasticsearch 的資料來做視覺化 (發 http request)
- Logstash
  - 處理 log (或其他資料，如訂單) 後把資料送給 Elasticsearch (也可以傳給不同的對象，如 Kafka)
  - 資料處理的 pipeline
  - pipeline 三步驟:
    - input
    - filter
    - output
  - pipeline configuration 的格式像 json
- X-Pack
  - 提供額外功能給 Elasticsearch 和 Kibana
  - Security
    - authentication
    - authorization
  - Monitoring
    - 視覺化 Elastic Stack 如何運作 (如顯示 CPU / memory usage / disk space)
  - Alerting
    - 設定告警 (如透過 email 通知)
  - Reporting
    - 輸出 Kibana 的資料
  - Machine Learning
    - enable machine learning for Elasticsearch & Kibana
  - Graph
    - 分析資料的關聯
  - Elasticsearch SQL
    - 可以傳送 SQL query 給 Elasticsearch
  - Beats
    - 蒐集資料並運送

## Walkthrough of common architectures
- 同時儲存資料在 DB 和 Elasticsearch，後端收到 request 的時候兩邊都做 query
