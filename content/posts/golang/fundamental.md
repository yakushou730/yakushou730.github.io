---
title: "fundamental"
authors: [yakushou730]
date: 2021-12-27T16:10:27+08:00
description: "fundamental"
tags: ["programming","golang"]
draft: false
---

## in memory
程式的用法在記憶體上會切成很多小區塊，每一塊我們稱為 word

32 bits CPU: 1 word = 32bits = 4 bytes
64 bits CPU: 1 word = 64bits = 8 bytes

## 如何在 go get 的時候可以拉到 private repo
概念:

因為是 private repo 的關係， 所以要讓 go 有權限去拉

拉的時候是透過 git，所以要設定具有權限的 token 給 git

1. 在個人的 github settings 建立一個可以拉取 private repo 的 token
2. 把 token 記下以後，在本機操作 git cli
3. 用 copy 下來的 token 取代下列的 GITHUB_TOKEN

```shell
$ git config --global url."https://${GITHUB_TOKEN}@github.com".insteadOf "https://github.com"
```

## Type Assertion vs Type Conversion
Type Assertion (斷言)
- 從 interface object 直接斷言型態是什麼
```go
var greeting interface{} = "hello world"
greetingStr := greeting.(string)
```

Type Conversion (型態轉換)
- 原本就是 concrete type，做型別轉換
```go
greeting := []byte("hello world")
greetingStr := string(greeting)
```

## Type switch
如果要確認 variable 的 type 的話，可以用 type switch
```go
switch v := param.(type) { 
default:
    fmt.Printf("Unexpected type %T", v)
case uint64:
    fmt.Println("Integer type")
case string:
    fmt.Println("String type")
}
```

## empty struct 的作用
當我們想要節省記憶體空間的時候用空結構

空結構不需要花費記憶體給 value (即不用 value)

```go
a := struct{}{}

// 印出結果會是 0
println(unsafe.Sizeof(a)) 
// 0
```

1. 當實作 data set 的時候
```go
map_obj := make(map[string]struct{})
for _, value := range []string{"interviewbit", "golang", "questions"} {
map_obj[value] = struct{}{}
}
fmt.Println(map_obj)
// Output
map[interviewbit:{} golang:{} questions:{}]
```

2. In graph traversals in the map of tracking visited vertices.
```go
visited := make(map[string]struct{})
for _, isExists := visited[v]; !isExists {
    // First time visiting a vertex.
    visited[v] = struct{}{}
}
```

3. 當 channel 需要訊號，而不是需要資料的時候
```go
func workerRoutine(ch chan struct{}) {
    // Receive message from main program.
    <-ch
    println("Signal Received")

    // Send a message to the main program.
    close(ch)
}

func main() {
    //Create channel
    ch := make(chan struct{})
    //define workerRoutine
    go workerRoutine(ch)
    // Send signal to worker goroutine
    ch <- struct{}{}
    // Receive a message from the workerRoutine.
    <-ch
    println(“Signal Received")
}
```

## 如何 copy slice 和 map
用 `copy()` 複製整個 slice
```go
slice1 := []int{1, 2}
slice2 := []int{3, 4}
slice3 := slice1
copy(slice1, slice2)
fmt.Println(slice1, slice2, slice3)
// Output
[3 4] [3 4] [3 4]
```

用 `=` 只重新賦予 slice description
```go
slice1 := []int{1, 2}
slice2 := []int{3, 4}
slice3 := slice1
slice1 = slice2
fmt.Println(slice1, slice2, slice3)
# Output
[3 4] [3 4] [1 2]
```

copy map 的話只能用 map 遍歷來做
```go
map1 := map[string]bool{"Interview": true, "Bit": true}
map2 := make(map[string]bool)
for key, value := range map1 {
	map2[key] = value
}
```

只是要 copy map 的 description 的話 用 `=`
```go
map1 := map[string]bool{"Interview": true, "Bit": true}
map2 := map[string]bool{"Interview": true, "Questions": true}
map3 := map1
map1 = map2    //copy description
fmt.Println(map1, map2, map3)
```

## GOROOT vs GOPATH
GOROOT:
- 放置官方的程式庫
GOPATH:
- 放置第三方套件，還沒有 go module 以前的 workspace

```shell
ls $GOPATH
# bin pkg src
```

**go run**
- 將程式碼編譯，產生可執行檔，運行該可執行檔，運行後系統自動刪除
 
**go build**
- 編譯出執行檔

## 如何對 custom struct 排序
要用到 `sort.Sort()`

需要實作以下 interface
```go
type Interface interface {
    // Find number of elements in collection
    Len() int
    
    // Less method is used for identifying which elements among index i and j are lesser and is used for sorting
    Less(i, j int) bool
    
    // Swap method is used for swapping elements with indexes i and j
    Swap(i, j int)
}
```

舉例
```go
type Human struct {
    name string
    age  int
}

// AgeFactor implements sort.Interface that sorts the slice based on age field.
type AgeFactor []Human
func (a AgeFactor) Len() int           { return len(a) }
func (a AgeFactor) Less(i, j int) bool { return a[i].age < a[j].age }
func (a AgeFactor) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }

func main() {
    audience := []Human{
        {"Alice", 35},
        {"Bob", 45},
        {"James", 25},
    }
    sort.Sort(AgeFactor(audience))
    fmt.Println(audience)
}

// Output
[{James 25} {Alice 35} {Bob 45}]
```

## byte vs rune
`byte` 是 `uint8` 的 alias，可代表 ASCII 字元

`rune` 是 `int32` 的 alias，可代表 UTF-8 編碼

rune literal 是單引號刮起來的 `'a'`

## list 倒轉
用兩個索引，一個從前面 一個從後面，做 `num1, num2 = num2, num1`
```go
package main

import "fmt"

func swapContents(listObj []int) {
        for i, j := 0, len(listObj)-1; i < j; i, j = i+1, j-1 {
                listObj[i], listObj[j] = listObj[j], listObj[i]
        }
}
func main() {
	listObj := []int{1, 2, 3}
	swapContents(listObj)
	fmt.Println(listObj)
}
// Output
[3 2 1]
```

## 如何有效率的對字串做 concatenate
要使用 `strings.Builder`

```go
package main

import (
    "strings"
    "fmt"
)

func main() {
    var str strings.Builder

    for i := 0; i < 5; i++ {
        str.WriteString("a")
    }

    fmt.Println(str.String())
}
// Output
aaaaa
```

## package 載入時的執行順序
`import --> const --> var --> init()`

如果 import 多個 package 的時候，按照 package 順序以上面的順序執行

每個 imported package 初始完以後再往下一個 package 執行


## value & reference
value type
- 透過 `=` 做 assign 給另外一個變數的時候，值的 copy 會被放在另一個變數的 memory
- value type 的變數會被放在 stack memory

reference type
- 複雜的資料，使用上會佔用很多 word 的則會用 reference type
- 指向複雜資料的第一個 word 
- assign reference type 的時候，只有存放的 address 被 copy 了
- 指向的資料改了的話，全部指向同一個記憶體位址的資料都會一起被改動
- 指向的資料是被放在 memory heap 上的，會碰到 gc，且記憶體空間也比 stack 大得多

## int
int 的儲存大小取決於 CPU 架構
- 32 bits CPU : 32 bits (4 bytes)
- 64 bits CPU : 64 bits (8 bytes)

int8, int16, int32, int64: 這類的是指定儲存大小

## float
多用 float64，因為 math package 的 function 都使用此 type

## 進位制表示
octal notation (8進制) : prefix 放 `0`
- 63 表示為 077

hexadecimal notation (16進制) : prefix 放 `0x`
- 255 表示為 0xFF

scientific notation (科學表示) : 用 `e` 表示 10 的次方
- 1000 可以寫成 1e3

## Character type
嚴格來說不是 golang 的 type，而是 int 的特殊情境

byte: alias of `uint8`
- 1 byte 可以描述 ASCII 的字元
- 例 `'A'` 是 `65` (10進位) 或 `\x41` (16進位)

> '\x' 後面一定是接兩個數字，是16進位的表示 如: '\x41'
> 
> '\' 後面一定是接三個數字，是8進位的表示 如: '\377'

Character 也支援 Unicode (UTF-8)

Character 稱為 Unicode code points

一個 unicode character 是用一個 int 存在記憶體位址，表示為 `U+hhhh` (h 是 16進位表示)

rune: alias of `int32`

要用 unicode-character 的話，16 進制前要用 `\u` 或 `\U`
- `\u` 是 4 位 hex digits
- `\U` 是 8 位 hex digits

## bitwise operators
AND: `&`

OR: `|`

XOR: `^`

CLEAR `&^`

COMPLEMENT: `^`

## 運算子優先順序
1st: `^` `!`
2nd: `*` `/` `%` `<<` `>>` `&` `&^`
3rd: `+` `-` `|` `^`
4th: `==` `!=` `<` `<=` `>=` `>`
5th: `<-`
6th: `&&`
7th: `||`

> 加上 `()` 的話會變成最優先


## string
string 是一連串的 UTF-8 characters (each 1 to 4 bytes)
- variable-width
- immutable arrays of bytes

> the 1-byte ASCII-code is used when possible, and a 2-4 byte UTF-8 code when necessary
> 
> len() to string is the number of bytes
> 
> taking the address of a character in a string is illegal

## pointer
`&`: address-of operator
`*`: type modifier

pointer 的大小 (都是一個 word)
- 32 bits machine -> 4 bytes
- 64 bits machine -> 8 bytes

> 透過 pointer refer 到 value，稱為 indirection
> 
> pointer 的 default 是 nil

透過 `*` 取指標的值稱為 dereference

參數用指標傳的好處是 如果是很大的結構的話就不用傳 copy of value，可節省記憶體使用

只要還有一個指標指向該記憶體，就不會被清掉，所以生命週期是獨立於建立時的 scope 之外

因為指標是 indirection，所以在非必要時使用的話 會造成效能下降

nil 的指標無法被 dereference，會 panic

## Reference
[Golang Interview Questions](https://www.interviewbit.com/golang-interview-questions/)
