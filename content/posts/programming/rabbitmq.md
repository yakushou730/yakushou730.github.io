---
title: "rabbitmq"
authors: [yakushou730]
date: 2022-02-08T00:49:12+08:00
description: "rabbitmq"
tags: ["programming"]
draft: false
---

[RabbitMQ](https://www.rabbitmq.com/)

RabbitMQ = message broker (accepts, stores, and forwards binary blobs of data - messages)

術語:
- Producer: 送出 messages (sending)
- Queue: a large message buffer
    - producer 可以送 message 到 queue 裡面
    - consumer 可以從 queue 把 message 收走
- Consumer: 等待接收 messages (receiving)

> producer, consumer, broker 不需要在相同的 host 上

要先安裝 rabbitMQ service [安裝連結](https://www.rabbitmq.com/download.html)

以下是用 **golang** 實作練習

用到的 package [amqp091-go](https://github.com/rabbitmq/amqp091-go)

## Hello World
實作兩個檔案
- send.go (代表 producer 送出 message)
- receive.go (代表 consumer 接收 message)

**實作 send.go**

設定 rabbitMQ 連線資訊 (socket connection)
```go
// 設定 rabbitMQ 連線資訊
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
// defer 把 conn close 掉
defer conn.Close()
```

建立通訊 channel
```go
// 建立通訊 channel
ch, err := conn.Channel()
// defer 把 channel close 掉
defer ch.Close()
```

必須宣告要使用哪個 queue 來 send 資料，再透過 publish 把資料送進 queue
```go
q, err := ch.QueueDeclare(
  "hello", // name
  false,   // durable
  false,   // delete when unused
  false,   // exclusive
  false,   // no-wait
  nil,     // arguments
)

body := "Hello World!"
err = ch.Publish(
  "",     // exchange
  q.Name, // routing key
  false,  // mandatory
  false,  // immediate
  amqp.Publishing {
    ContentType: "text/plain",
    Body:        []byte(body),
  })
```

注意建立 queue 是 idempotent 的，只有在對應的 queue 不從在的時候才會建立

message 是 byte array 的形式，所以可以傳送任何資料

**實作 receive.go**

連線的地方和 send 設定一樣
- open connection
- open channel
- 宣告要用哪個 queue (確保通道一定存在，不存在的話會建立)

```go
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
defer conn.Close()

ch, err := conn.Channel()
defer ch.Close()

q, err := ch.QueueDeclare(
  "hello", // name
  false,   // durable
  false,   // delete when unused
  false,   // exclusive
  false,   // no-wait
  nil,     // arguments
)
```



