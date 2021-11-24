---
title: "errors"
authors: [yakushou730]
date: 2021-11-23T18:26:55+08:00
description: "errors"
tags: ["golang"]
draft: false
---

## #As
`func As(err error, target interface{}) bool`

第一個參數是現在拿到的 error

會透過呼叫 **Unwrap** 來比對 err 的第一個錯誤是否和 target 相同
如果對的話，會把 target 設定為 err 並回傳 true
否則回傳 false

範例
```go
err := dec.Decode(dst)
if err != nil {
	var syntaxError *json.SyntaxError
	switch {
	case errors.As(err, &syntaxError):
		return fmt.Errorf("body contains badly-formed JSON (at character %d)", syntaxError.Offset)
	default:
		return err
	}
}
```

## #Is
`func Is(err error, target error) bool`

第一個參數是現在拿到的 error
第二個參數是想要比對的特定 error

比對方式是透過呼叫 **Unwrap** 逐一拆解來比對是否包含 特定的 error

範例
```go
err := bcrypt.CompareHashAndPassword(p.hash, []byte(plaintextPassword))
if err != nil {
	switch {
	case errors.Is(err, bcrypt.ErrMismatchedHashAndPassword):
		return false, nil
	default:
		return false, err
	}
}
```
