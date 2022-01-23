---
title: "Refactor With Bill"
authors: [yakushou730]
date: 2022-01-22T13:46:18+08:00
description: "Refactor With Bill"
tags: ["programming","golang"]
draft: false
---

## Project init
結構會是先建出下列資料夾 (一個 repo 可以執行多個 binary 的結構)

再來是做 go mod init [name]
```
# 可執行的程式入口
cmd/violin/css/
cmd/violin/image/
cmd/violin/mp3/
cmd/violin/templates/
cmd/violin/internal/
cmd/violin/main.go

# 商業邏輯
internal/platform/
go.mod
```

起手式 main.go
```go
func main(){
	if err := run(); err != nil {
		log.Println(err)
		os.Exit(1)
    }
}

func run() {
	return nil
}
```
