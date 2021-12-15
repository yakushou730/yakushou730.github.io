---
title: "getting started"
authors: [yakushou730]
date: 2021-12-14T10:38:32+08:00
description: "getting started"
tags: ["udemy", "elasticsearch"]
draft: false
---

## Installation
- 安裝 Elasticsearch
- 安裝後可以用 `http://localhost:9200` 確認服務是否正常
- Elasticsearch 的檔案目錄
    - bin
      - 放置執行檔 (Elasticsearch 以外的還有如 plugin 安裝檔 或 cli)
    - config
      - 放置 configuration file
      - elasticsearch.yml
        - 檔案內容是 comment 標起來的，如果不另外把 comment 拿掉的話，系統會使用預設值
        - best practice 是要註明 cluster.name 和 node.name
        - data / log 的路徑最好也自行指定 (特別指在 production 上)
    
          這樣即使砍了 Elasticsearch 也不會有資料遺失
        - jvm.options
          - 用來調整系統的記憶體配置
        - log4j2.properties
          - 寫 log 的 framework
    - data
    - jdk
      - 包含 OpenJDK
    - lib
      - 包含 Elasticsearch 會用到的 dependency
    - logs
    - modules
      - 包含很多內建的 module
    - plugins

## Basic architecture
- 基礎單元是 node，是 Elasticsearch 的 instance，用來儲存 data
- node 可以執行複數個，各自儲存部分的資料
- node 不是機器，所以可以用幾台機器運行更多個 node
- 每個 node 都隸屬在 cluster 下
- cluster 包含 nodes
- 通常不同的 cluster 會用來做不同的搜尋情境，彼此獨立
- 資料單位稱為 document，是一筆 json 格式的資料
- index a document，意思是送一筆 json object 給 Elasticsearch
  - 該筆 json object 到 Elasticsearch 後會在 `"_source"` 的節點下，並賦予其他 meta data 
- index 包含多個 document
- query 的時候，是對指定的 index 做搜尋 document

## Kibana console
- 進入 Kibana 介面，並點擊 Dev Tools
- query
  - `HTTP method` + `request path` 

範例: 取得 cluster 狀態
```
# _cluster: API
# health: Command
GET /_cluster/health
```

回傳結果
```json
{
  "cluster_name" : "elasticsearch_yakushou730",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 8,
  "active_shards" : 8,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

範例: 取得 nodes 資訊
```
# _cat: 輸出為人好辨識的格式
# nodes: Command
# v: 增加顯示敘述的 header
GET /_cat/nodes?v
```

回傳結果
```
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
127.0.0.1           54         100  15    2.45                  cdfhilmrstw *      cengyaoshangdeMacBook-Pro.local
```

範例: 取得更多 node 資訊
```
GET /_nodes 
```

範例: 查看更多 index 資訊
```
GET /_cat/indices?v
```

結果
```
health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases                _z2mYFpaTam1bSmP3zjR_g   1   0         42            0     41.1mb         41.1mb
green  open   .apm-custom-link                A08XOda6TK-A7rD5IEccRA   1   0          0            0       226b           226b
green  open   .kibana_task_manager_7.16.0_001 6J6EJiuIRSW-8gkU1yS7LQ   1   0         17        99299      9.9mb          9.9mb
green  open   .apm-agent-configuration        str7doPtT6mphioHd4qVOg   1   0          0            0       226b           226b
green  open   .kibana_7.16.0_001              l2dmfJcEQ0me1B6tLsYDiw   1   0        462            2      2.7mb          2.7mb
```

## Sending queries with cURL

可以直接透過 Kibana 介面，選擇 copy as cURL

`Content-Type` 很重要，需要是 `application/json`

如果是對 Elastic Cloud 的話，多加上 `-u account:password`

## Sharding and scalablility

- Sharding 用來把 index 切成小塊，每一小塊都稱作 `shard`
- 主要是因為可水平拓展
- 可以把單一 index 的資料分散到不同的 node
- 可以在不同的 node 平行搜尋

範例:
```
# 列出 indices
GET /_cat/indices?v

# 結果
health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases                _z2mYFpaTam1bSmP3zjR_g   1   0         42            0     41.1mb         41.1mb
green  open   .apm-custom-link                A08XOda6TK-A7rD5IEccRA   1   0          0            0       226b           226b
green  open   .kibana_task_manager_7.16.0_001 6J6EJiuIRSW-8gkU1yS7LQ   1   0         17       185141     18.8mb         18.8mb
green  open   .apm-agent-configuration        str7doPtT6mphioHd4qVOg   1   0          0            0       226b           226b
green  open   .kibana_7.16.0_001              l2dmfJcEQ0me1B6tLsYDiw   1   0        466            9      2.4mb          2.4mb
```

上面欄位 pri 表示 primary shards

- 7.0.0 版本後，index 只有 1 個 default 的 single shard
- 透過 split API 增加 shard 數量
- 透過 shrink API 減少 shard 數量

## Replication

- configured at index level
- 建立 copies of shards，作為 replica shards
- primary shard 複製出來的副本，不會放在同相同的 node 
- Elasticsearch 提供 snapshot 做為備份
- snapshot 可以備份 index level 或整個 cluster
- snapshot vs replication
  - snapshot 是用來做固定時間的備份
  - replication 是在某個節點掛掉之後一樣可以撈到資料
- 一個 replication group 內的 replica shards 可以平行做搜尋 
- Elasticsearch 預設對每個 index 會有一個 replica，但如果沒有多個 node 的話就不會被分配空間
- replication group 是指 primary shard + primary shard 的 replicas
- 增加一個 index，default 會有 1 primary shard 和 1 replica shard
