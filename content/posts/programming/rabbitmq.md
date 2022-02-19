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

message 會是從 channel ( from `amqp::Consume` ) 以 asynchronously 的方式收到

開一個 goroutine 來接收

```go
msgs, err := ch.Consume(
  q.Name, // queue
  "",     // consumer
  true,   // auto-ack
  false,  // exclusive
  false,  // no-local
  false,  // no-wait
  nil,    // args
)

forever := make(chan bool)

go func() {
  for d := range msgs {
    log.Printf("Received a message: %s", d.Body)
  }
}()

log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
<-forever
```

## Work Queues (aka: Task Queue)
會有多數個 consumer 來接收 queue 送出來的 message

實作兩個檔案
- new_task.go (代表 producer 送出 message)
- worker.go (代表 consumer 接收 message)

**實作 new_task.go**
```go
// 連線的地方上前個練習一樣
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
defer conn.Close()

ch, err := conn.Channel
defer ch.Close()

q, err := ch.QueueDeclare(
    "task_queue", // name
    true,         // durable
    false,        // delete when unused
    false,        // exclusive
    false,        // no-wait
    nil,          // arguments
)
```

傳送整理過後的資訊
```go
body := bodyFrom(os.Args)
err = ch.Publish(
	"",     // exchange
	q.Name, // routing key
	false,  // mandatory
	false,
	amqp.Publishing{
		DeliveryMode: amqp.Persistent,
		ContentType:  "text/plain",
		Body:         []byte(body),
    })
log.Printf(" [x] Sent %s", body)
```

其中 bodyForm 用來把 arg 參數串成一個字串
```go
func bodyFrom(args []string) string {
    var s string
    if (len(args) < 2) || os.Args[1] == "" {
        s = "hello"
    } else {
        s = strings.Join(args[1:], " ")
    }
    return s
}
```

**實作 worker.go**

```go
// 連線的地方上前個練習一樣
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
defer conn.Close()

ch, err := conn.Channel()
defer ch.Close()

q, err := ch.QueueDeclare(
"task_queue", // name
    true,         // durable
    false,        // delete when unused
    false,        // exclusive
    false,        // no-wait
    nil,          // arguments
)
```

調整要接收資料的地方
```go
err = ch.Qos(
	1,     // prefetch count
	0,     // prefetch size
	false, // global
)

msgs, err := ch.Consume(
	q.Name, // queue
	"",     // consumer
	false,  // auto-ack
	false,  // exclusive
	false,  // no-local
	false,  // no-wait
	nil,    // args
)

forever := make(chan bool)

go func() {
	for d := range msgs {
		log.Printf("Received a message: %s", d.Body)
		dotCount := bytes.Count(d.Body, []byte("."))
		t := time.Duration(dotCount)
		time.Sleep(t * time.Second)
		log.Printf("Done")
		d.Ack(false)
	}
}()

log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
<-forever

```
