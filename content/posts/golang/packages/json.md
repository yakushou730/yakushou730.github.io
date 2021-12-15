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

## omitempty

這個欄位 如果是結構的話

無論是否為空值，一定會產生出欄位，值為空

例如
```go
s := struct {
	Foo struct {
        Bar string `json:",omitempty"`
	} `json:",omitempty"`
}{}

// encoding result
{"Foo":{}}
```

解決辦法是透過指標的方式

例如
```go
s := struct {
    Foo *struct {
        Bar string `json:",omitempty"`
    } `json:",omitempty"`
}{}

// encoding result
{}
```

此外如果欄位型態是 `time.Time` 的話

omitempty 不會運作

## escaped

json 編碼的時候如果碰到 `>` `<` `&`

會轉為逃逸字元

範例
```go
s := []string{
	"<foo>",
    "bar & baz",
}

// result
["\u003cfoo\u003e","bar \u0026 baz"]
```

如果要避免的話，要在 `json.Encoder` 實例使用 `SetEscapeHTML(false)`

