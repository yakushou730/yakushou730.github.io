---
title: "Web Development"
authors: [yakushou730]
date: 2021-11-17T16:45:33+08:00
description: "Web Development"
tags: ["programming"]
draft: false
---

## web url encoding
- 在 url 內的 `+` 代表空白，也可以用 `%20` 表示

## user enumeration

一種可能的帳號攻擊行為

比如說嘗試使用者登入，輸入email 和 密碼

系統顯示 密碼錯誤 而不是 帳號密碼錯誤 或 找不到此使用者

代表這個使用者的 email 信箱是存在且有註冊過這個網站的

即使用者的資訊被其他人知曉

要避免的話，可以統一錯誤訊息或是把錯誤訊息寫得更通用一點

## CORS

cross-origin requests

origin: 如果兩個 URL 的 scheme, host, port 都相同，

即為相同 origin

(path 不需要相同)

> https://foo.com:443/a/
> - https 是 scheme
> - foo.com 是 host
> - 443 是 port
> - a 是 path

- web browser 預設是禁止的
- 網站可以內嵌其他 origin 的資源
- 一個網站可以傳送資料給不同的 origin 位址
- 但是一個網站不能夠接收來自不同 origin 的資料 (瀏覽器行為)
- 可以透過設定 Header 內的 `Access-Control` 來控制允許或不允洗特定的 cross-origin requests 打我們的 API
- 藍覽器以外的工具，如 curl, wget 不會受到影響

> 如果在 API 的 response header 的 Header 的 `Access-Control-Allow-Origin` 塞值 `*`
> 
> 即表示告訴 接受 response 的 browser 說這個 API 的資料是所有網站都可以拿的 
> 
> 如果只想給特定的網站拿資料，那就把對應的網址填入 `Access-Control-Allow-Origin` 的值 

Simple 類型的 CORS
- 為 HEAD, GET, POST 之一
- Headers 都不為 forbidden headers，或以下 CORS-safe 其中之一
  - Accept
  - Accept-Language
  - Content-Language
  - Content-Type
- 如果有 Content-Type 的話，值要是以下之一
  - application/x-www-form-urlencoded
  - multipart/form-data
  - text/plain

如果不為以上的 Simple 類型，瀏覽器會在真正的 request 之前觸發 preflight request

用意是決定這個真正的 CORS 是否可以被允許

Preflight 類型的 request (三要件)
- HTTP method OPTIONS
- `Origin` Header
- `Access-Control-Request-Method` Header

而系統在收到 preflight request 的時候需要加在 response 的 Header
- `Access-Control-Allow-Origin` 值放 preflight request 的 `Origin
- `Access-Control-Allow-Methods` 列出可以用來給真正 CORS request 的 HTTP methods
- `Access-Control-Allow-Headers` 列出可以被包含在真正 CORS request 的 Headers
