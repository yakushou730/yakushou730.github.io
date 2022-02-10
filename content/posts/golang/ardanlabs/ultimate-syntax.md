---
title: "ultimate syntax"
authors: [yakushou730]
date: 2022-01-18T08:25:04+08:00
description: "ultimate syntax"
tags: ["programming","golang"]
draft: false
---

## Variables
- 一般來說使用 int 較多，而不是 int32 或 int64
- 少用 `:= zero value`，碰到 zero value 的話不如直接用 var 宣吿

## Strings
- string 是 2 words 的 data structures
  - 空字串
    - 第一個 word 是 `nil`
    - 第二個 word 是 `0` 
  - 字串 Hello
    - 第一個 word 是指標，指向 `hello` 陣列的 `h` 位址 
    - 第二個 word 是長度 `5`

## Struct
golang 的三種印出格式
- `%v` -> `{false 0 0}`
- `%+v` -> `{flag:false counter:0 pi:0}`
- `%#v` -> `main.example{flag:false, counter:0, pi:0}`

**Literal Construction**
```go
e2 := example{
	flag:    true,
	counter: 10,
	pi:      3.141592
}
```

**Empty Literal Construction**

不建議，因為在某些情境下不會真的都是 zero value
```go
e2 := example{}
```

建議如果是 zero value 的話就用 `var`
```go
var e2 example
```

如果 struct 裡面有 zero value 的欄位，建立的時候不用 assign value

不要做 partial construction
```go
// 不要做 partial construction
var e example
e.flag = true
// 而是
e := example{
	flag: true
}
```

## Type Conversions
struct literal type 可以 assign 給 name type

但不同的 name type 彼此間不能互相 assign

```go
type bill struct{
	flag bool
}

type eric struc{
	flag bool
}

e := struct{
	flag bool
}{
	flag: true
}

var bi bill
var er eric

// compile failed
bi = er

// compile succeed
bi = e

// compile succeed
bi = bill(er)
```

## Pointers
- value semantics: a piece of code is copied as moved in your program
- pointer semantics: one copy and shared by everybody

value semantics 可以讓資料彼此都是獨立的 (isolated) 不會碰到 mutation bug

> data integrity: 數據完整性

pointer semantics 的話 因為太多人可以改他，所以 data integrity 很差


## Literal Struct
是指沒有 name 的 type

不是什麼東西都要給一個 name (type pollution)

```go
e2 := struct {
	flag bool
	counter int16
	pi float32
}{
	flag:    true,
	counter: 10,
	pi:      3.141592
}
```

## Constants
```go
// untyped constants
const ui = 12345     // kind: integer
const uf = 3.141592  // kind: floating-point

// typed constants
const ti int = 12345         // type: int
const tf float64 = 3.141592  // type: float64

// Variable answer will of type float64
var answer = 3 * 0.333 // KindFloat(3) * KindFloat(3.0)

// Constant third will be of kind floating point
const third = 1 / 3.0 // KindFloat(1) / KindFloat(3.0)

// Constant zero will be of kind integer
const zero = 1 / 3 // KindInt(1) / KindInt(3)

// Max Integer value on 64 bit architecture
const maxInt = 9223372036854775807

// Much larger value than int64
// compile 階段不會報錯
const bigger = 9223372036854775808543522345
// 會錯 overflows，因為指定了型別
const biggerInt int64 = 9223372036854775808543522345
```

iota 只有在 block 內有作用
```go
const (
	A = iota // 0
	B        // 1
	C        // 2
)

const (
	Ldate         = 1 << iota //  1
	Ltime                     //  2  
	Lmicroseconds             //  4
	Llongfile                 //  8
	Lshortfile                // 16
	LUTC                      // 32
)
```

> 不要因為想要 compiler protection 而建立多餘的 type
> 
> 維持 type 的一致性

error 是 interface type

## Literal Functions

> Functions are values
> 
> Functions are first class values in go

```go
// Declare an anonymous function and call it
// 呼叫的當下，n 是什麼就是什麼
func() {
	fmt.Println("Direct:", n)
}()

// 更常見的用法是宣告匿名 function，並賦值給變數
f := func(){
	fmt.Println("Variable:", n)
}
// 透過變數呼叫 function
f()
```

## Array Basics

宣告 Array 大小的時候不能用變數

```go
// ... 表示完全依照後面的項目決定數量，合法語法但不常用
numbers := [...]int{10, 20, 30, 40}
```

用 range 方式 iterate 的 item 是 copy
```go
func main() {
	arr := [5]int{1, 2, 3, 4, 5}
	for _, v := range arr {
		v += 1
	}
	fmt.Println(arr)
}

// result
[1 2 3 4 5]
```
