---
title: "fundamental1"
authors: [yakushou730]
date: 2021-12-27T16:10:27+08:00
description: "fundamental1"
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

## switch
switch 有一種用法是在 switch 語句的時候做 initialization statement

> 這種用法要在 init 後段手動加上 `;`

```go
switch a, b := x[i], y[j]; {
    case a < b: t = -1
    case a == b: t = 0
    case a > b: t = 1
}
```

## loop
不要在 loop 的 body 裡面更改 counter variable

初始化多於一個變數的範例
```go
for i, j := 0, N; i < j; i, j = i+1, j-1 { }
```

> 用 for i 的方式來 loop 非 ASCII 字串的話，會看到依照 byte 被切開的非預期結果

`for ix, val := range coll {}` 的語法中 ix 是 index，而 val 是 coll 在 index 處的 copy
- 即 val 只能作為 only read-purposes

> 勁量不要用 goto
> 
> 若需要使用 goto LABEL 時，對應的 LABEL 最好是在程式後段
> 
> 往前跳的話會讓 code 很難讀

## functions
3 種類型
- Normal functions with an identifier
- anonymous or lambda functions
- methods

> function signature: function 的 參數 和 回傳值 (包含 type)
> 
> called / invoked 是指對 function 的呼叫
> 
> 當呼叫 function 的時候，參數會以 copy 的形式被傳進 function

function 在 golang 中可以是 first-class object，即作為參數傳送或作為 return value

或指派給變數

> function 可以是一個 type 作為宣告，zero value 是 nil
> 
> function value 可以做比較，如果都是指向同一個 function 的話就是 true
> 
> 或都是 nil 的話為 true

名詞定義:
```go
greeting(lastname)

func greeting(name string)  {}
```

lastname 是 `actual parameter`

name 是 `formal parameter`

> niladic function: 指不需要參數的 function，如: main.main()

function 的傳入參數
- by value: 傳入 value 的 copy
- by reference: 傳入 address 的 copy
  - 可以變更記憶體位址對應的值
  - 和 copy 物件相比，copy reference 的成本較低
  - `slice` `map` `interface` `channel` 預設上是 pass by reference

**defer**

defer call 是 function scoped

常見用法
- 關閉檔案串流
- 解鎖已上鎖資源 (mutex)
- 印出報表的 footer
- 關閉資料庫連線

**built-in function**
- close: 關閉 channel communication
- len & cap
  - len: `strings, arrays, slices, maps, channels` 的長度
  - cap: `slices, maps` 的容量
- new & make: 用來配置記憶體位址的
  - new: 用在 value types 和 user defined types (structs)
    - new(T) 回傳的是記憶體位址，對應的值為 zero value
  - make: 用在 built-in reference types `slices, maps, channels`
    - make(T) 回傳出使用的變數 type T
- copy & append
  - copy: copying slices
  - append: concatenating slices
- panic & recover: 作為 error handling 用的
- print & println: 低階的輸出方式，一般使用 fmt package 輸出即可
- complex & real & imag: 用作複數計算

**Higher-Order functions**
```go
func inc1(x int) int { return x+1 }
f1 := inc1
```

舉例: 把函式當參數傳遞的例子
```go
// func IndexFunc(s string, f func(c int) bool) int
strings.IndexFunc(line, unicode.IsSpace)
```

> tips: 其他例子可以為 第一個參數給 raw data，第二個參數給 function 來處理前面的 raw data
> 
> 像是 第一個參數給 int slice，第二個參數給判斷 `isOdd(n int) bool` 或 `isEven(n int) bool` 的 function

匿名函式
- anonymous function
- 又稱 lambda function, function literal, closure
- 例: `func(x, y int) int { return x + y }`
- 無法自行存在，但可以被指派給變數 `fplus := func(x, y int) int { return x + y }`

因為匿名函式可以在內部設定共用參數，所以可以用另一種非遞迴的寫法來實作 Fibonacci
```go
func fib() func() int {
    a, b := 1, 1
    return func() int {
        a, b = b, a + b
        return b
    }
}
```

**Factory function**

當 function 可以回傳 function 的時候，也可以作為 factory function 使用

例
```go
func MakeAddSuffix(suffix string) func(string) string {
    return func(name string) string {
        if !strings.HasSuffix(name, suffix) {
            return name + suffix
        }
        return name
    }
}

// 使用
addBmp := MakeAddSuffix(".bmp")
addJpeg := MakeAddSuffix(".jpeg")

addBmp("file") // returns: file.bmp
addJpeg("file") // returns: file.jpeg
```

**Memoization**

可增加效能: 對於已經做過的運算就不要重複做，而是重複利用 (透過記在 memory 做 cache 的方式)

尚未理解的呼叫次序
```go
package main
import "fmt"
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}
func un(s string) {
    fmt.Println("leaving:", s)
}
func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}
func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}
func main() {
    b()
}

// Results
//
// entering: b
// in b
// entering: a
// in a
// leaving: a
// leaving: b
```

尚未理解呼叫次序第二彈
```go
func trace(name string) func() {
	fmt.Printf("Entering %s\n", name)
	return func() {
		fmt.Printf("Leaving %s\n", name)
	}
}
func main() {
	defer trace("f")()
	fmt.Println("Doing something")
}
// Results
//
// Entering f
// Doing something
// Leaving f
```

## Arrays and Slices
Slice 是建立在 Array 上的抽象層

array 的 length 需要明確，因為在 compile 時要分配記憶體空間

> The maximum array length is 2GB

array 是 golang 的 `value type`，所以可以透過 `new` 來建立

```go
// 下面的寫法有差異

// new 所回傳的是記憶體位址，所以 arr2 是 *[5]int，是不為 nil 的記憶體位址
var arr2 = new([5]int)

// var 的話是 type [5]int
var arr1 [5]int
```

> array 的賦值是 value copy，不會影響到原本的 array
> 
> 傳 array 作為參數到 function 時也是傳 copy

`[...]int{1,2,3}` 的寫法，compiler 會自己去算個數，還是 array

但不能用 `[...]int` 作為 type

array 宣告時可以指定第幾個位置要放什麼內容
```go
var arrKeyValue = [5]string{3: "Chris", 4: "ron"}
```

> 當傳送很大的 array 到 function 的時候會吃很多記憶體
> 
> 因為 array 傳入 function 時是 copy by value
> 
> 解法
> - 傳 array 的指標
>   - 傳進去的 array 不需要做 dereference 即可直接使用
> - 使用 slice of the array

slice 是 reference type，所以不需要額外的記憶體位空間，使用起來比 array 有效率

slice 在 memory 內是三個欄位的結構
1. 一個 pointer 指向 underlying array
2. slice 的 length
3. slice 的 capacity

> slices are **indexable** and have a length given by the `len()` function

slice 的程度是可以動態變動的
- 最小是 0
- 最大是 underlying array 的長度

對於 slice 來說 `cap()` 指的是這個 slice 的長度可以變成多長
- 是 `slice 的長度` 加上 `array 超過 slice 的長度`

> s 是 slice
> 
> cap 是指從 s[0] 開始到 array 的最後
> 
> slice 的 length 絕不會超過 capacity
> 
> `0 <= len(s) <= cap(s)`

相同 underlying array 的不同 slices 間，可以分享資料

不同的 array 是不能分享資料的

```go
// 這樣宣告出來的 slice 指向 nil
// len 0
var identifier []type
// slice 宣告的時候可以指定是哪個 array 的切片 (start 到 end - 1)
var slice1 []type = arr1[start:end]
// slice 宣告的時候指定整個 array
var slice1 []type = arr[:]
// 其他切片語法
s := [3]int{1,2,3}[:]
s := [...]int{1,2,3}[:]
s := []int{1,2,3}
var x = []int{1,2,3,4,5}
// 會參照相同的 underlying array
s2 := s[:]
// 以下為 true
s == s[:i] + s[i:]
// 可以做移位操作
s2 = s2[1:]
```

傳參數時，使用 slice 會比傳 array 來得有效率

```go
slice1 := make([]type, len)
slice1 := make([]type, len, cap)
// 以下是等價的
make([]int, 50, 100)
new([100]int)[0:50]
```

`new()` vs `make()`
- 兩個都是在 heap 上分配記憶體位址，但做不同的事，用到的 type 也不一樣
- new -> allocates
- make -> initializes
- `new(T)`
  - array, struct
  - 建立 0 值，並回傳 address
- `make(T)`
  - slice, map, channel
  - 回傳初始化 type T 的值

> arrays, slices 一直都是 1-dimensional 的
> 
> 透過 compose 的方式做成更高的維度

```go
values := [][]int{} // multidimensional slice
// These are the first two rows.
row1 := []int{1, 2, 3}
row2 := []int{4, 5, 6}

// Append each row to the two-dimensional slice.
values = append(values, row1)
values = append(values, row2)
```

**`bytes` package**

byte slice 在 bytes package 很常見
```go
// New Buffer.
var b bytes.Buffer
// Write strings to the Buffer.

b.WriteString("ABC")
b.WriteString("DEF")
// Use Fprintf with Buffer.
fmt.Fprintf(&b, " A number: %d, a string: %v\n", 10, "bird")
b.WriteString("[DONE]")
// Convert to a string and print it.
fmt.Println(b.String())

// Outputs
ABCDEF A number: 10, a string: bird
[DONE]
```

> 透過 buffer 來操作字串連接，和 `+=` 比起來，
> 
> 在大量的字串連接上可以更有效使用 memory 和 CPU

```go
// ix 是 index
// value 是每一次 loop 的值，只做用在 body 內，是 copy of the slice item
// 所以不能用來更改 slice 內容
for ix, value := range slice1 {
...
}

// 要改內容的話要透過 index
seasons := []string{"Spring","Summer","Autumn","Winter"}
for ix := range seasons {
  seasons[ix] = strings.ToUpper(seasons[ix])  // modifying the seasons
}
```

```go
// 多擴充一位給 slice
sl = sl[0:len(sl)+1]
```

> slice 可以被 resize 直到 underlying array 滿了

增大 slice capacity 的方法是先建立一個更大的 slice 以後把資料 copy 過來

```go
// copy function
// 用 src 覆蓋掉 dst
// 回傳複製了多少 element
// 當 src 是 string 的時候，element type 是 byte
func copy(dst, src []T) int

// append function
// 如果 append 會造成超過 underlying type 的話，會分配到另一個新的且足夠大的 slice
func append(s[]T, x ...T) []T

// 把一個 slice append 到另一個 slice
x = append(x, y...)
```

> A string in memory is, in fact, a 2 word-structure consisting of a pointer to the string data and the length.

```go
// s 是 string
c := []byte(s)

// copy 也可以處理字串來源
copy(dst []byte, src string)

// range unicode string 的話
// 每一個 loop 是一個 rune
s := "\u00ff\u754c"
for i, c := range s {
    fmt.Printf("%d:%c ", i, c)
}
// Outputs
0:ÿ 2:界


// 建立 substring
substr := str[start:end]
```

string 是 immutable 的，不能直接指定替換 string 的元素

另一種方式是 先把 string 轉成 byte array 後變更，再轉回 string

```go
s := "hello"
c := []byte(s) // converting string s to array of bytes `c`.
c[0] = 'c'     // modifying the c 
s2 := string(c) // Converting c to string, s2 == "cello"
```

一些透過 append 做到的 slice 操作技巧
```go
// 把 b 加到 a
a = append(a, b...)
// 刪除 index i
a = append(a[:i], a[i+1:]...)
// 從 index i 到 j 的部分切掉
a = append(a[:i], a[j:]...)
// 使用新的 slice 做擴展
a = append(a, make([]T, j)...)
// 插入 item 到 index i
a = append(a[:i], append([]T{x}, a[i:]...)...)
// 插入新的 長度 j 的 slice 於 index i
a = append(a[:i], append(make([]T, j), a[i:]...)...)
// 插入已存在的 slice b 於 index i
a = append(a[:i], append(b, a[i:]...)...)
// 從 stack 拿掉最高的元素
x, a = a[len(a)-1], a[:len(a)-1]
// 插入一個元素到 stack
a = append(a, x)
```

> 在數學應用上 slice 又稱為 vector
> 
> 有需要的話可以做 alias 使用

Garbage collection 探索案例
```go
var digitRegexp = regexp.MustCompile("[0-9]+")
func FindDigits(filename string) []byte {
  b, _ := ioutil.ReadFile(filename)
  return digitRegexp.Find(b)
}
```

上述的程式有缺點，因為 slice `[]byte` 會用到的 underlying array 非常大 (整份 file)

因為被 reference 的關係會一直存在記憶體不會被清掉

所以要特別處理，免得記憶體爆炸

改善版本如下，建立一個需要用到的 slice (符合需求的 underlying array)

這樣就不會佔用額外記憶體空間

```go
func FindDigits(filename string) []byte {
  b, _ := ioutil.ReadFile(filename)
  b = digitRegexp.Find(b)
  c := make([]byte, len(b))
  copy(c, b) // copying b to c
  return c
}
```

```go
// string 可以直接做成 rune slice
str := "hello"
runes := []rune(str)
// 也可以直接做成 byte slice
bytes := []byte(str)

// 轉回 string
string(runes)
string(bytes)
```

## Maps
又稱為 associative arrays 或 dictionaries

```go
var map1 map[keytype]valuetype
// example
var map1 map[string]int
```

map 是 reference type

未初始化的 map value 是 `nil`

key type 可以是任何支援 `==` 和 `!=` 的 type
- 可以: string, int, float, array, struct, pointer, interface
- 不行: slice (因為 slice 沒有定義 equality)

> map 是 reference type，所以傳入 function 的成本很低
> 
> 查詢的效能: 透過 slice or array 的 index > 透過 map 的 key > 透過線性的 slice or array

map 沒取到值的話會是 zero value

`len(map1)` 可以取得這個 map 有多少對 key value

```go
// 初始化 map
var map1 map[keytype]valuetype = make(map[keytype]valuetype)
// or
map1 := make(map[keytype]valuetype)
```

value 使用 function 的範例
```go
func main() {
  mf := map[int]func() int{ // key type int, and value type func()int
    1: func() int { return 10 },
    2: func() int { return 20 },
    5: func() int { return 50 },
  }
  fmt.Println(mf)
}
// Outputs
// 注意 value 是 function 的 address
map[1:0x4011c0 2:0x4011d0 5:0x4011e0]
```

map 宣告的時候可以指定 capacity，如果後來超過的話，會逐漸增加 (+1,+1 這樣繼續長)
- 64 bits 的機器 default 是 8 bytes
```go
make(map[keytype]valuetype, cap)
```

```go
// 確認 map 是否有值
if _, ok := map1[key1]; ok {
  // ...
}

// 刪除 map 內的 key 值
delete(map1, key1)
```

可以在 range map 內刪除 map 的 key value pair

建立 map 的 slice
```go
items := make([]map[int]int, 5)
for i := range items {
  items[i] = make(map[int]int, 1) 
  items[i][1] = 2 // This 'item' data will not be lost on the next iteration
}
```

## packages
- `unsafe`
  - 一般不會用到，可以跳過 type 檢查
  - 和 C/C++ 交互作用時會用到
- `os`
  - 提供 os 層級的功能
  - unix-like design
- `os/exec`
  - 提供 執行外部 os 指令和程式
- `syscall`
  - low-level, external package, 針對 underlying OS's 呼叫提供原始的介面
- `archive/tar` 和 `archive/zip` 和 `compress`
  - 包含壓縮 / 解壓縮的功能
- `fmt`
  - 包含格式化輸入輸出的功能
- `io`
  - 提供基本 io 功能，大部分是包裝 os 指令
- `bufio`
  - 提供 buffered input/output 功能的包裝
- `path/filepath`
  - 包含 os 會用到的操作檔名路徑的程序
- `flag`
  - 包含 command-line 參數的功能
- `strings`
  - 包含操作和處理字串的功能
- `strconv`
  - 把字串轉成基本的資料型態
- `unicode`
  - 包含 unicode 的特殊函式
- `regexp`
  - 提供字串的 pattern 搜尋功能
- `bytes`
  - 包含操作 byte slice 的功能
- `index/suffixarry`
  - 包含在字串內非常快速搜尋的功能
- `math`
  - 包含基本數學常數和函式
- `math/cmplx`
  - 提供操作複數的 method
- `math/rand` 
  - 提供產生 pseudo-random 數字
- `sort`
  - 提供針對 array 和 user-defined collections 的排序功能
- `math/big`
  - 包含更高精度的數學方法
- `container`
  - 實作容器來操作 collections
    - `list`: 處理 doubly-linked list
    - `ring`: 處理 circular lists
- `time`
  - 包含處理 times, dates 的基本功能
- `log`
  - 包含處理程式執行中的 log 資訊
- `encoding/json`
  - 實作 JSON 格式的 讀取/解碼 和 寫入/編碼
- `encoding/xml`
  - 提供簡單的 XML 1.0 轉換
- `net`
  - 包含基本的 處理 network data
- `http`
  - 包含 parse HTTP requests/replies 的功能，提供 HTTP server 和基本的 client
- `html`
  - 提供 HTML5 的 parser
- `crypto - encoding - hash - ...`
  - 提供大部分的 package encrypting 和 decrypting
- `runtime`
  - 包含和 go-runtime 互動的操作，像是 garbage collection 和 go-routine
- `reflect`
  - 實作 runtime introspection，允許程式操作任意型別的變數 

`container/list` 範例
```go
func insertListElements(n int)(*list.List){   // add elements in list from 1 to n
  lst := list.New()
  for i:=1;i<=n;i++{
    lst.PushBack(i)         // insertion here
  } 
  return lst
}

func main() {
  n := 5                    // total number of elements to be inserted
  myList := insertListElements(n) // function call
  for e := myList.Front(); e != nil; e = e.Next() {
		fmt.Println(e.Value)  // printing values of list
	}
}
```

`regexp` 範例
```go
func main() {
  searchIn := "John: 2578.34 William: 4567.23 Steve: 5632.18" // string to search
  pat := "[0-9]+.[0-9]+" // pattern search in searchIn

  f := func (s string) string {
    v, _ := strconv.ParseFloat(s, 32)
    return strconv.FormatFloat(v * 2, 'f', 2, 32)
  }
  // 檢查是否 pattern 可以找到 match
  if ok, _ := regexp.Match(pat, []byte(searchIn)); ok {
    fmt.Println("Match found!")
  }
  
  // 做出指定 pattern 的 regexp object
  // 如果是用 MustCompile 的話，如果 pattern 轉換成 regexp object 失敗會 panic
  re, _ := regexp.Compile(pat)

  // 透過 regexp object 去處理輸入的 文章字串
  // 並用 ReplaceAllString 去做取代
  str := re.ReplaceAllString(searchIn, "##.#") // replace pat with "##.#"
  fmt.Println(str)
  // using a function
  str2 := re.ReplaceAllStringFunc(searchIn, f)
  fmt.Println(str2)
}
```

發生 race condition 的話，經典的解法是只讓一個 thread 可以更改 shared variable

shared variable 在程式內稱為 critical section，透過 lock 的機制來處理

同時只讓一個 thread 可以執行這段，當執行完 critical section 再 unlock

> map 沒有 internal locking 機制 (考慮到效能因素)
> 
> map type 不是 thread-safe
> 
> 所以 concurrent 存取會對 shared map 的資料造成資料毀損

處理 race condition 可以用到 `sync` package 的 `Mutex` (mutual exclusion lock)

用來保護 critical section，使得一次只有一個 thread 可以進入 critical section

範例
```go
// 假設 Info 是個要被保護的 shared variable
// 可以給他一個 sync.Mutex 變數來處理
type Info struct {
  mu sync.Mutex
  // ... other fields, e.g.:
  Str string
}

func Update(info *Info) {
    info.mu.Lock()
    // critical section:
    info.Str = // new value
    // end critical section
    info.mu.Unlock()
}
```

`RWMutex`
- 允許多個 reader threads 使用 `RLock()`，但一個 write thread

`once.Do(call)`
- `once` 是 type Once 的變數
- 不管呼叫到多少次，once 都只會執行一次 function call

`math/big`: 可以處理任意位數，唯一的限制是機器的可用記憶體大小
- `big.Int` 處理很大的整數
- `big.Rat` 處理有理數
- `big.Float`

`math/big` 範例
```go
func main() {
  // Here are some calculations with bigInts:
  im := big.NewInt(math.MaxInt64)
  in := im
  io := big.NewInt(1956)
  ip := big.NewInt(1)
  ip.Mul(im, in).Add(ip, im).Div(ip, io)
  fmt.Printf("Big Int: %v\n", ip)
  // Here are some calculations with big rationals:
  rm := big.NewRat(math.MaxInt64, 1956)
  rn := big.NewRat(-1956, math.MaxInt64)
  ro := big.NewRat(19, 56)
  rp := big.NewRat(1111, 2222)
  rq := big.NewRat(1, 1)
  rq.Mul(rm, rn).Add(rq, ro).Mul(rq, rp)
  fmt.Printf("Big Rat: %v\n", rq)
}
```

自訂 package 名稱
- short
- single-word
- lowercase
- filenames without `_`

`GOPATH`: 告訴 go 要去哪邊尋找和安裝 go package，是 workspace 的路徑
- `src`: 包含目錄結構的 go source files (.go)
- `pkg`: 包含安裝的和編譯的 package objects (.a)
- `bin`: 包含可執行檔 (.out 或 .exe)

`GOROOT`: golang 在系統中的 SDK (Software Development Kit) 位置

`go install importpath`: build 和安裝 package object (.a)
- 路徑 `pkg/linux_amd64\book` (linux)

一個檔案可以有不只一個 `init()` 就會是隨機順序
- init() 只會在 initialize 階段呼叫一次

`go doc [package name]` 可以印出 package 的文件
- `godoc -http=:6060` 可以在 localhost:6060 以 html 開啟 doc

跑全部的測試 `go test ./...`

`go install` vs `go get`
- `go get` 驗證 package 是否需要被下載，是的話會下載並編譯
- `go install` 不會下載，只會編譯

## Structs and Methods
struct 是 value type，且可以用 `new()` 建立，內含的欄位稱為 `fields`

golang 並沒有 class 的概念，所以 struct type 很重要

`t := new(T)`
- 在物件導向會稱呼 t 為 struct T 的 instance 或 object
- 在 golang 會稱呼 t 為 struct T 的 value
- `new(Type)` 和 `&Type{}` 是等價的 (equivalent expression)

可透過 `size := unsafe.Sizeof(T{})` 查看 struct T 的一個 value 佔用了多少記憶體 (bytes)

```go
// 這樣宣告是兩個相同 underlying type 的不同 type，不可直接互用，要轉型
type nr number   // new distinct type
// 這樣宣告是可互通的 不同 type
type nrAlias = number // alias type
```

**Factory Method**
像物件導向的建構子的用法，如 `func NewFile(fd int, name string) *File`

要怎麼避免呼叫的人直接用 `new()` 的方法作為建構 struct，就是把那個結構做成 unexposed struct

這樣 `func NewXXX() *XXX` 就會變成唯一的建構方式

struct 內欄位用到的 tag 只有 `reflect` package 可以存取

tag 範例
```go
type TagType struct { // tags
    field1 bool "An important answer"
    field2 string `The name of the thing`
    field3 int "How much there are"
}

tt := TagType{true, "Barack Obama", 1}

for i:= 0; i < 3; i++ {
    ttType := reflect.TypeOf(tt)
    ixField := ttType.Field(i)       // getting field at a position ix
    fmt.Printf("%v\n", ixField.Tag)   // printing tags
}
```

取得 tag value 範例
```go
type T struct {
  a int "This is a tag"
  b int `A raw string tag`
  c int `key1:"value1" key2:"value2"`
}

if field, ok := reflect.TypeOf(t).FieldByName("c"); ok {
    fmt.Println(field.Tag)
    fmt.Println(field.Tag.Get("key1"))
}
```

在 struct 內，一樣 type 的 anonymous filed 只能有一個

結構內相同欄位名稱的話，會從外層先呼叫，再往內層找

> Methods are not mixed with the data definition (the structs).
> 
> They are orthogonal to types;
> 
> representation (data) and behavior (methods) are independent.

Pointer and value methods can both be called on the pointer or non-pointer values.

method 和 function 轉換的範例
```go
type T struct {
  a int
}

func (t T) print(message string) {
  fmt.Println(message, t.a)
}

func (T) hello(message string) {
  fmt.Println("Hello!", message)
}

func callMethod(t T, method func(T, string)) {
  method(t, "A message")
}

func main() {
  t1 := T{10}
  t2 := T{20}
  var f func(T, string) = T.print
  callMethod(t1, f)
  callMethod(t2, f)
  callMethod(t1, T.hello)
}
```

convention:
- 取得欄位的 method 直接用大寫開頭命名該欄位名稱
- 設定欄位的 method 加上 `Set` 的 prefix

> `(recv T) String() {}` 的 body 不能使用 fmt 作為字串串接，否則會發生 infinite loop

embedded type 的做法很像物件導向的繼承，也很像 ruby 的 mixin
- 可以拿到 embedded type 的 method 來用

範例
```go
type Point struct {
  x, y float64
}

func (p *Point) Abs() float64 {
  return math.Sqrt(p.x*p.x + p.y*p.y)
}

type NamedPoint struct {
  Point   // anonymous field of Point type
  name string
}

func main() {
  n := &NamedPoint{Point{3, 4}, "Pythagoras"} // making pointer type variable
  fmt.Println(n.Abs()) // prints 5
}
```

2 種 embed 的方式
1. Aggregation (or composition) : 包含 named field of the type of the wanted functionality
2. Embedding: 鑲嵌 wanted type anonymously

golang 可以透過 embedded 的方式達到 繼承複數個父類別的概念

在少數情況，可以呼叫 `runtime.GC()` 來呼叫 garbage collection

讀取 memory status 的範例

```go
func main() {
  ms := runtime.MemStats{}
  runtime.ReadMemStats(&ms)
    
  println("Heap after GC. Used:", ms.HeapInuse, " Free:", ms.HeapIdle, " Meta:", ms.GCSys)

  time.Sleep(5 * time.Second)
}
```

如果當要把某個 object 從記憶體移除時同步呼叫某個行為 (例如寫 log)

可以透過以下方式呼叫來達成

`runtime.SetFinalizer(obj, func(obj *typeObj))`

當程式正常結束或發生錯誤的時候 SetFinalizer 不會被執行

當對應的 object 要被 GC 清掉的時候才會執行

## interface
interface 格式

```go
// 名稱通常是以 -er 為結尾
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
// 宣告，此時 ai 尚未初始化，值是 nil
// interface 是 reference type
var ai Namer
// ai 是 receiver value + method table ptr
```

interface 已經是 reference type，所以不會有 pointers to interface

- type 在實作介面的時候不需要明確的闡明是哪個 interface，只要 method set 滿足就自動成立
- 多個 type 都可以實作相同的 interface
- 實作 interface 的 type 仍然可以有其他 function
- 一個 type 可以實作多個 interface
- interface 對應到一個實作該介面的 instance 的 reference

> interface 就是 golang 的 polymorphism
> 
> 要注意實作 interface 介面的是 struct object type instance 或 struct pointer type instance

範例
```go
// io.Reader interface
type `Reader` interface {
    Read(p []byte) (n int, err error)
}

var r io.Reader
// 以下都有滿足介面 Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
f,_ := os.Open("test.txt")
r = bufio.NewReader(f)
```

interface 內部還可以內嵌其他 interface
```go
type ReadWrite interface {
  Read(b Buffer) bool
  Write(b Buffer) bool
}

type Lock interface {
  Lock()
  Unlock()
}

type File interface {
  ReadWrite
  Lock
  Close()
}
```

interface 型態的變數可以透過 assertion 判斷是不是特定的 concrete type
```go
// 判斷 interface type 是不是 concrete type T
if v, ok := varI.(T); ok { // checked type assertion
    Process(v)
    return
}
// here varI is not of type T
```

> Note: Always use the comma, ok form for type assertions

**Type Switch**

要注意 `Fallthrough is not permitted`

用到 `element.(type)` 的 assertion 判斷不要用在 switch 以外的地方

範例
```go
switch t := areaIntf.(type) {
    case *Square:
      fmt.Printf("Type Square %T with value %v\n", t, t)
    case *Circle:
      fmt.Printf("Type Circle %T with value %v\n", t, t)
    default:
      fmt.Printf("Unexpected type %T", t)
}
```

> 透過 type 判斷來做什麼事 非常有幫助，像是 parse JSON or XML data

assertion 也可以判斷是否 concrete type 滿足 interface
```go
type Stringer interface { String() string }

if sv, ok := v.(Stringer); ok {
  fmt.Printf("v implements String(): %s\n", sv.String()); // note: sv, not v
}
```

> a pointer can be dereferenced for the receiver

透過 interface 呼叫 method 的時候，一定要是 identical receiver type 或是 直接可是別的 concrete type
- pointer methods can be called with pointers
- value methods can be called with values
- value-receiver methods can be called with pointer values (because they can be dereferenced first)
- pointer-receiver methods `cannot` be called with values; because the value store inside an interface has no address

`io` package 提供了 `io.Reader` 和 `io.Writer` 介面
```go
type Reader interface {
  Read(p []byte) (n int, err error)
}

type Writer interface {
  Write(p []byte) (n int, err error)
}
```

- Read() 是指 read from
  - 一個 object 要能夠是 readable 的話，要實作 `io.Reader`
  - `Read()` method 讀回物件的 data，並放到 byte array 中
    - 即 把資料讀到 `p []byte` 裏面 (用 p []byte 去裝資料回來)
    - n 是裝了多長的資料
    - 沒發生錯誤的話，err 為 `nil` 或 `io.EOF`
- Write() 是指 write to
  - 一個 object 要能夠是 writable 的話，要實作 `io.Writer`
  - `Write()` 是把 byte array 的資料寫道物件裡面
- io 提供的 Reader 和 Writer 是 unbuffered 的
  - `bufio` 提供了對應的 buffered 操作
    - 在 讀取/寫入 UTF-8 encoded 文件時非常有用

**Empty Interface**

`interface{}` 支援了所有型別
- 每個 `interface{}` 變數佔用了 2 words 的記憶體
  - 1 word for type
  - 1 word for either the contained data or pointer to it

透過 interface{} 的作法，可以做出裝不同 type 的 slice
```go
type Element interface{}

type Vector struct {
  a []Element
}
```

`[]interface{}` 要裝東西的話不能直接 assign

可以用 for loop 針對每一個 item 做 assign

**reflect package**

reflection 主要是透過 type 來檢驗 structure 的能力

另一個稱呼是 meta-programming

reflect 可以在 runtime 時調查 type 和 variable
- size
- methods
- 可以動態呼叫這些方法
- 小心使用，非必要時儘量避免使用

> Basic information of a variable is its type and its value:
> 
> these are represented in the reflection package by the types Type, which represents a general Go type,
> 
> and Value, which is the reflection interface to a Go value.
> 
> `reflect.TypeOf` `reflect.ValueOf`

`Kind()` 可以用來拿到 type
```go
var x float64 = 3.4
v := reflect.ValueOf(x)
// v 是 reflect.Float64
```

透過 refect 去 set value 有很多限制，這邊僅列出可動的範例
```go
func main() {
  var x float64 = 3.4
  v := reflect.ValueOf(x)
  // setting a value:
  // Error: will panic: reflect.Value.SetFloat using unaddressable value
  // v.SetFloat(3.1415)
  fmt.Println("settability of v:", v.CanSet())
  v = reflect.ValueOf(&x) // Note: take the address of x.
  fmt.Println("type of v:", v.Type())
  fmt.Println("settability of v:", v.CanSet())
  v = v.Elem()
  fmt.Println("The Elem of v is: ", v)
  fmt.Println("settability of v:", v.CanSet())
  v.SetFloat(3.1415) // this works!
  fmt.Println(v.Interface())

  fmt.Println(v)
}
// outputs
settability of v: false
type of v: *float64
settability of v: false
The Elem of v is:  3.4
settability of v: true
3.1415
3.1415
```

> 透過 struct 可以將 `map[string]interface{}` 轉成 struct
> 
> 但複雜且不常用，故略過

範例
```go
func mapToStruct(m map[string]interface{}) interface{} {
	var structFields []reflect.StructField

	for k, v := range m {
		sf := reflect.StructField{
			Name: strings.Title(k),
			Type: reflect.TypeOf(v),
		}
		structFields = append(structFields, sf)
	}

	// Creates the struct type
	structType := reflect.StructOf(structFields)

	// Creates a new struct
	structObject := reflect.New(structType)

	return structObject.Interface()
}


func main() {

	m := make(map[string]interface{})

	m["name"] = "Barack"
	m["surname"] = "Obama"
	m["age"] = 57

	s := mapToStruct(m)
	fmt.Printf("%t %[1]v\n", s)

	sr := reflect.ValueOf(s)
	sr.Elem().FieldByName("Name").SetString("Donald")
	sr.Elem().FieldByName("Surname").SetString("Trump")
	sr.Elem().FieldByName("Age").SetInt(72)
	fmt.Println(s)
}
```

當一個 struct 裡面包含了 另一個 struct (實作了多數介面) 的指標，

則那個 struct 可以呼叫 embedded struct 的 interface methods

如
```go
type Task struct {
  Command string
  *log.Logger
}
// New
func NewTask(command string, logger *log.Logger) *Task {
  return &Task{command, logger}
}
// task 可以呼叫 Log 的方法
task.Log()
```

在 struct 內嶔 interface

不要搞混，這是指在 struct 裡面有對應的 field 可以塞滿足 interface 的物件
```go
type ReaderWriter struct {
  io.Reader
  io.Writer
}
// 初始化

t := &ReaderWriter{
    Reader: nil, // 可以塞滿足 interface 的物件
    Writer: nil,
}
```

當把 slice 也做成一種 type 的時候，就可以對該 type 做新的 method
```go
type Cars []*Car
// Cars 可以有方法
func (cs Cars) Process(f func(car *Car)) {
    for _, c := range cs {
        f(c)
    }
}
```

> interface 包含了 value 和 type
> 
> 只有在 type 和 value 都是 nil 的情況下才會是 nil
