---
title: "Signal"
authors: [yakushou730]
date: 2021-11-20T23:24:11+08:00
description: "Signal"
tags: ["programming","golang"]
draft: false
---

## Signal 類型

Signal|Description|Keyboard Shortcut|Catchable 
---|---|---|---  
SIGINT|鍵盤中斷訊號|Ctrl + C|true
SIGQUIT|鍵盤離開訊號|Ctrl + \|true
SIGKILL|刪除程序(立即終止)|-|no
SIGTERM|照順序終止程序|-|true


> `kill pid` 會傳送SIGTERM到程序pid

## shutdown()

shutdown 的關閉流程不會中斷任何作用中的連線
1. 先關閉所有 open 的 listener
2. 再關閉所有 idle 的 listener
3. 等候作用中連線成為 idle
4. shutdown

http.Server 的 Shutdown 會在結束時回傳錯誤碼 (沒錯誤的話回傳 nil)

## 關閉 server 架構流程
1. 另外開一個 go routine 用 signal.Notify 監控是否收到終止訊號
2. 如果收到終止訊號的話，呼叫 http.Server#Shutdown，
   並把回傳值塞到監聽 shutdown 結果的 channel
3. main 在 http.Server#ListenAndServe 結束後，
   檢查回傳結果是否為 ErrServerClosed
4. 如果是的話，等候監聽 shutdown 結果的 channel 回傳資料
5. 若監聽 shutdown 的 channel 回傳 nil，代表已經關閉成功
