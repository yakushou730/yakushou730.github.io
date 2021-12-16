---
title: "managing documents"
authors: [yakushou730]
date: 2021-12-15T22:57:51+08:00
description: "managing documents"
tags: ["udemy", "elasticsearch"]
draft: false
---

## Creating & deleting indices

新增 index

範例: 新增 products index
```
PUT /products
# 結果
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "products"
}
```
- acknowledged: 表示是否有執行成功
- shards_acknowledged: 表示 required shards 是否有在 timeout 前 start up
  - default 是指 primary shard

刪除 index

範例: 刪除 products index
```
DELETE /products
```

## Indexing documents

插入資料: index a document

```
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}

# result
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "ek25vn0BLyjHJvg3PVzk",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```
_shards: 表示有多少 shards 成功/失敗 儲存這筆資料
_id: 表示這筆資料的 identifier (系統自動產生，或者也可以自己指定)

自己指定 _id 的情境
```
PUT /products/_doc/100
{
  "name": "Teaser",
  "price": 49,
  "in_stock": 4
}

# result
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 4,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```
表示要建立 _id: 100

## Retrieving documents by ID

```
GET /products/_doc/100

# result
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 7,
  "_seq_no" : 7,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Teaser",
    "price" : 49,
    "in_stock" : 4
  }
}
```
有找到的話 會有 _source 的 key

且 found 為 true

沒找到的話會沒有 _source 的 key 且 found 為 false

## Updating documents

```
POST /products/_update/100
{
  "doc": {
    "in_stock": 3  
  }
}

# result
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 8,
  "result" : "noop",
  "_shards" : {
    "total" : 0,
    "successful" : 0,
    "failed" : 0
  },
  "_seq_no" : 8,
  "_primary_term" : 1
}
```

result key 為 updated 表示有更新

如果沒更新的話 (例如資料一樣) 那值為 noop

- Elasticsearch documents are immutable
- 更新的時候看起來像 update change，但實際上是 replace (整個替換)

## Scripted updates

有點像是寫 script 的概念去更新 _source

直接指定要操作的欄位並更新

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}

# result
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 11,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 11,
  "_primary_term" : 1
}

# 其他範例
# 可指定 params 並做為參數使用
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock += params.quantity",
    "params": {
      "quantity": 10
    }
  }
}
```

不管有沒有資料更改，scripted updates 顯示的 result 都會是 updated
- 除非特定指定 `ctx.op = "noop"`

## Upserts

意思是如果資料不存在的話就建立

存在的話就更新

```
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"
  },
  "upsert": {
    "name": "Blender",
    "price": 399,
    "in_stock": 5
  }
}
```

第一次的話會建立 upsert block 內的資料

再執行一次的話 in_stock 會加 1

## Replacing documents

範例:
```
PUT /products/_doc/100
{
  "name": "Teaser",
  "price": 49,
  "in_stock": 4
}
```
不管原本 id 100 的內容是啥

這個寫法都會強行覆蓋掉

## Deleting documents

```
DELETE /products/_doc/101
```

## Understanding routing
Routing is the process of resolving a shard for a data

index 的 shard 不能改變的原因是因為 routing formula

## Understanding document versioning
- 不是版控，回不到過去資料
- `_version` 欄位

## Optimistic concurrency control

在更新欄位時加入 `if_primary_term` 和 `if_seq_no` 條件

這樣可以確保在要更新的時間點，是否當時的版本還是預期的

如果有其他人更新過，那版本會對不起來，更新失敗

## Update by query
使用一次 query 更新多筆 document

範例: 每個 in_stock 都減 1
```
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}

# result
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "products",
        "_type" : "_doc",
        "_id" : "ek25vn0BLyjHJvg3PVzk",
        "_score" : 1.0,
        "_source" : {
          "price" : 64,
          "name" : "Coffee Maker",
          "in_stock" : 8
        }
      },
      {
        "_index" : "products",
        "_type" : "_doc",
        "_id" : "100",
        "_score" : 1.0,
        "_source" : {
          "price" : 49,
          "name" : "Test",
          "in_stock" : 2
        }
      }
    ]
  }
}
```

驗證更新
```
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
```

## Delete by query
依條件刪除資料

```
POST /products/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

## Batch processing
一個 query 更新多筆資料 - Bulk API

支援 index / create / update / delete

create 和 index 的差異
- create: 若資料已存在會建立失敗
- index: 若資料已存在會 replace，不存在則建立

```
POST /_bulk
{ "index": { "_index": "products", "_id": 200 }}
{ "name": "Espresso Machine", "price": 199, "in_stock": 5 }
{ "create": { "_index": "products", "_id": 201 }}
{ "name": "Milk Frother", "price": 149, "in_stock": 14 }
```

驗證都用
```
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
```

更新和刪除
```
POST /_bulk
{ "update": { "_index": "products", "_id": 201 }}
{ "doc": { "price": 129 }}
{ "delete": { "_index": "products", "_id": 200 }}
```

上面那行在 index 相同的情況又可以寫成
```
POST /products/_bulk
{ "update": { "_id": 201 }}
{ "doc": { "price": 129 }}
{ "delete": { "_id": 200 }}
```

- bulk API 的 HTTP request Header 要是 `Content-Type: application/x-ndjson`
- 每行的結尾都要是 `\n` 或 `\r\n` 
- 如果是文字編輯器的話，最後一行要是空行
- 一個動作失敗不會影響到其他動作
- 透過 `items` key 來判斷動作是否成功
- 支援 optimistic concurrency control

## Importing data with cURL

準備一份 products-bulk.json file (最後一行要空白行)

```json
{"index":{"_id":1}}
{"name":"Wine - Maipo Valle Cabernet","price":152,"in_stock":38,"sold":47,"tags":["Alcohol","Wine"],"description":"Aliquam augue quam, sollicitudin vitae, consectetuer eget, rutrum at, lorem. Integer tincidunt ante vel ipsum. Praesent blandit lacinia erat. Vestibulum sed magna at nunc commodo placerat. Praesent blandit. Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede. Morbi porttitor lorem id ligula.","is_active":true,"created":"2004\/05\/13"}
{"index":{"_id":2}}
{"name":"Tart Shells - Savory","price":99,"in_stock":10,"sold":430,"tags":[],"description":"Pellentesque at nulla. Suspendisse potenti. Cras in purus eu magna vulputate luctus. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Vivamus vestibulum sagittis sapien. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Etiam vel augue. Vestibulum rutrum rutrum neque. Aenean auctor gravida sem.","is_active":true,"created":"2007\/10\/14"}
{"index":{"_id":3}}

```

指令
```shell
$ curl -H "Content-Type: application/x-ndjson" -X POST http://localhost:9200/products/_bulk --data-binary "@products-bulk.json" 
```

可以在 Kibana 下指令驗證 shard 的資料變多了
```
GET /_cat/shards?v
```
