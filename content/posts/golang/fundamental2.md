---
title: "fundamental2"
authors: [yakushou730]
date: 2022-03-06T22:05:26+08:00
description: "fundamental2"
tags: ["programming","golang"]
draft: false
---

## Reading and Writing

`bufio` 用於 buffered input 和 output

最簡單的 input 方式是使用 fmt 的 Scan 方法

```go
var firstName, lastName string
fmt.Scanln(&firstName, &lastName)
```

`Scanln` 從 standard input 讀取 text，以空白隔開的輸入，直到掃到 newline

`Scanf`, `Fscanf`, `Sscanf` 透過 format string 來 parse 參數
- `Scanf` 透過 keyboard 輸入
- `Sscanf` 透過其他 string 輸入

透過 `bufio` 取得輸入字串
```go
// os.Stdin 滿足 io.Reader
inputReader = bufio.NewReader(os.Stdin)
fmt.Println("Please enter some input: ")
input, err = inputReader.ReadString('\n')
// err 是 io.EOF 如果讀到結尾的話
```

os.Stdout 是指印在 screen 上

`bufio` 允許從任何滿足 Reader 的 type 來讀取輸入

**Reading from a file**

在 golang，Files 是透過 `os.File` 的物件指標，也叫 file handles
- os.Stdin 和 os.Stdout 都是 *os.File 類型

標準的讀取檔案範例
```go
func main() {
  inputFile, inputError := os.Open("input.dat")
  if inputError != nil {
    fmt.Printf("An error occurred on opening the inputfile\n" +

    "Does the file exist?\n" +
    "Have you got access to it?\n")
    return // exit the function on error
  }
  defer inputFile.Close()
  inputReader := bufio.NewReader(inputFile)
  for {
    inputString, readerError := inputReader.ReadString('\n')
    if readerError == io.EOF {
      return
  }
    fmt.Printf("The input was: %s", inputString)
  }
}
```

讀取檔案的另一個簡單範例
```go
func data(name string) string {
  f := os.Open(name, os.O_RDONLY, 0)
  defer f.Close() // idiomatic Go code!
  contents := io.ReadAll(f)
  return contents
}
```

讀取檔案的另一個範例
```go
func main() {
  inputFile := "products.txt"
  // buf 讀出來是 []byte
  buf, err := ioutil.ReadFile(inputFile)
  if err != nil {
    fmt.Fprintf(os.Stderr, "File Error: %s\n", err)
  }
  fmt.Printf("%s\n", string(buf))
}
```

> 如果檔案很大的話要避免使用 ReadFile 做一次讀取，很吃記憶體

讀取檔案的另一個範例
```go
buf := make([]byte, 1024)
...
n, err := inputReader.Read(buf)
if (n == 0) { break}
// n 是取的 bytes 數量
```

`fmt.Fscanln()` 可以用來讀取檔案的每一行，並賦值給變數
```go
_, err := fmt.Fscanln(file, &v1, &v2, &v3) // scans until newline
```

> 檔案開啟後都要記得 `defer file.close()`
>
> `path` package 的 `filepaht` 提供 os platform 操作 檔名 和 路徑 的功能

可透過 `compress` package 讀取壓縮檔
- bzip2
- flate
- gzip
- lzw
- zlib

範例
```go
func main() {
    fName := "Example.json.gz"
    var r *bufio.Reader
    fi, err := os.Open(fName)
    if err != nil {
        fmt.Fprintf(os.Stderr, "%v, Can't open %s: error: %s\n", os.Args[0], fName, err)
        os.Exit(1)
    }
    fz, err := gzip.NewReader(fi)
    if err != nil {
        r = bufio.NewReader(fi)
    } else {
        r = bufio.NewReader(fz)
    }
    for {
        line, err := r.ReadString('\n')
        if err != nil {
            fmt.Println("Done reading file")
            os.Exit(0)
        }
        fmt.Println(line)
    }
}
```

**Writing data to a file**

範例
```go
func main () {
  outputFile, outputError := os.OpenFile("output/output.dat", os.O_WRONLY|os.O_CREATE, 0666)
  if outputError != nil {
    fmt.Printf("An error occurred with file creation\n")
    return
  }
  defer outputFile.Close()
  outputWriter:= bufio.NewWriter(outputFile)
  outputString := "hello world!\n"
  for i:=0; i<10; i++ {
    outputWriter.WriteString(outputString)
  }
  outputWriter.Flush()
}
```

`os.OpenFile` 提供除了開啟檔案以外更多的功能
- os.O_RDONLY: read-only
- os.O_WRONLY: write-only
- os.O_RDWR: both read and write
- os.O_CREATE: create the file if it doesn't exist
- os.O_TRUNC: to truncate to size 0 if the file already exists

另一種寫入檔案方式
```go
fmt.Fprintf(outputFile, "Some test data.\n")
```

上面的方法可以寫入任何的 `io.Writer`

另一個範例
```go
func main() {
  os.Stdout.WriteString("hello, world\n")
  f, _ := os.OpenFile("output/test.txt", os.O_CREATE|os.O_WRONLY, 0)
  defer f.Close()

  f.WriteString("hello, world in a file\n")
}
```

使用 `ioutil` package 讀寫檔案
```go
type Page struct {
	Title string
	Body  []byte
}

func (p *Page) save() error {
	filename := p.Title + ".txt"
	return ioutil.WriteFile(filename, p.Body, 0600)
}

func load(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := ioutil.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}
```

copy file 就是開啟兩個 file 檔案以後，透過 io.Copy 去做

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
  src, err := os.Open(srcName)
  if err != nil {
    return
  }
  defer src.Close()
  dst, err := os.OpenFile(dstName, os.O_WRONLY|os.O_CREATE, 0644)
  if err != nil {
    return
  }
  defer dst.Close()
  return io.Copy(dst, src)
}
```

**Reading arguments from the command-line**

`os.Args` 提供了 slice of string 來裝 command-line 引數
- `os.Args[0]` 是程式名稱
- os.Args[1:] 都是引數字串

`flag` package 專門用來處理 運行程式時的引數
- `-h` 會列出所有提供的引數
- `flag.Parse()` 呼叫後會轉換並設定好參數

```go
// iterate os.Args
var s string
for i := 0; i < flag.NArg(); i++ {
    s += flag.Arg(i)
}
```

注意 flag.Arg 和 os.Args 是不一樣的
- flag.Arg 是指轉換之後的參數

讀取檔案並印出
```go
func cat(f *os.File) {
  const NBUF = 512
  var buf [NBUF]byte
  for {
    switch nr, err := f.Read(buf[:]); true {
    case nr < 0:
      fmt.Fprintf(os.Stderr, "cat: error reading: %s\n", err.Error())
      os.Exit(1)
    case nr == 0: // EOF
      return
    case nr > 0:
      if nw, ew := os.Stdout.Write(buf[0:nr]); nw != nr {
        fmt.Fprintf(os.Stderr, "cat: error writing: %s\n")
      }
    }
  }
}
```

**Scanning inputs**

讀取 stdin 的資料
```go
scanner := bufio.NewScanner(os.Stdin)
fmt.Println("Please enters some input: ")
if scanner.Scan() {
  fmt.Println("The input was", scanner.Text())
}
```

另一個用 bufio 讀取輸入的範例
```go
func main() {
  scanner := bufio.NewScanner(os.Stdin)
  fmt.Println("Please enter some input: ")
  // scanner.Split(bufio.ScanRunes) // <-- scan runes (newline included)
  scanner.Split(bufio.ScanWords) // <-- scan words (sequence of characters delimited by spaces)
  // scanner.Split(bufio.ScanLines) // <-- the default: scan lines
  for scanner.Scan(){ // The for loop stops when a EOF is given
    fmt.Printf("Token ->%s<-\n", scanner.Text())
  }
}
```

> Never forget to use Flush() when terminating buffered writing, else the last output won’t be written.

**The JSON data format**

- data struct -> special format string
    - marshaling or decoding
- special format string to data strcut
    - unmarshaling or decoding

示意範例，把 data 轉成 json
```go
// 透過 json.Marshal
js, _ := json.Marshal(vc)
  
// 或
// 透過 Encoder
enc := json.NewEncoder(file)
err := enc.Encode(vc)
```

> 由於網路安全因素，使用 `json.MarshalForHTML()` 的話會把 `<script>` 內的字串做溢出

golang type whit JSON format
- `bool` for JSON booleans
- `float64` for JSON numbers
- `string` for JSON strings
- `nil` for JSON null

為了 Encode go map type，必須使用 `map[string]T`

只有 exported 的欄位會被 json package 存取

**Unmarshal**

```go
type FamilyMember struct {
    Name string
    Age int
    Parents []string
}
// b 是 byte slice 
var m FamilyMember
err := json.Unmarshal(b, &m)
```

**Decoding arbitrary data**

`json` package 使用 `map[string]interface{}` 和 `[]interface{}` 來存 JSON objects 和 arrays

範例
```go
b := []byte(`{"Name": "Wednesday", "Age": 6, "Parents": ["Gomez", "Morticia"]}`)
var f interface{}
err := json.Unmarshal(b, &f)
// f 變成
map[string]interface{}{
    "Name": "Wednesday",
    "Age": 6,
    "Parents": []interface{}{
        "Gomez",
        "Morticia",
    },
}
```

另一個 marshal / unmarshal 的範例
```go
type Person struct {
  Name string `json:"personName"`
  Age int `json:"personAge"`
}

func main() {
  b := []byte(`{"personName": "Obama", "personAge": 57}`)
  var p Person
  // Unmarshalling
  json.Unmarshal(b, &p)
  fmt.Println(p)
  // Marshalling
  js, _ := json.Marshal(p)
  fmt.Printf("%s\n", js)
}
```

**The XML Data format**

就像 json package 一樣，也支援 marshal / unmarshal

使用 `encoding/xml` package

```go
type Person struct {
  Name string `xml:"personName"`
  Age int `xml:"personAge"`
}

func main() {

  b := []byte(`<Person><personName>Obama</personName><personAge>57</personAge></Person>`)
  var p Person
  // Unmarshalling
  xml.Unmarshal(b, &p)
  fmt.Println(p)
  // Marshalling
  xmlString, _ := xml.Marshal(p)
  fmt.Printf("%s\n", xmlString)
}
```

> Unmarshal vs NewDecoder
> 
> 使用場景:
> 
> 如果是 in memory 的 data (ex: some user input) 使用 unmarshal
> 
> 如果是 streaming (ex: file) 使用 NewDecoder

> 暫不對 XML 多做紀錄，先跳過

**gob**

gob 是 golang 對資料做成 2進位制格式的 serializing / deserializing
- 不可跨語言

使用 `encoding` package

gob (short for Go binary format)

通常用在 remote procedure calls (RPCs)

或用在 application 和 machine 之間的通訊

範例
```go
type P struct {
  X, Y, Z int
  Name string
}

type Q struct {
  X, Y *int32
  Name string
}

func main() {
  // Initialize the encoder and decoder. Normally enc and dec would be
  // bound to network connections and the encoder and decoder would
  // run in different processes.
  var network bytes.Buffer // Stand-in for a network connection
  enc := gob.NewEncoder(&network) // Will write to network.
  dec := gob.NewDecoder(&network)// Will read from network.
  // Encode (send) the value.
  err := enc.Encode(P{3, 4, 5, "Pythagoras"})
  if err != nil {
    log.Fatal("encode error:", err)
  }
  // Decode (receive) the value.
  var q Q
  err = dec.Decode(&q)
  if err != nil {
    log.Fatal("decode error:", err)
  }
  fmt.Printf("%q: {%d,%d}\n", q.Name, *q.X, *q.Y)
}
```

> gob 等有常用的時候再補充，先跳過

**Cryptography with Go**

`hash` package
- adler32
- crc32
- crc64
- fnv

`crypto`
- md5
- sha1
- aes
- rc4
- rsa
- x509

範例
```go
func main() {
  hasher := sha1.New()
  io.WriteString(hasher, "test")
  b := []byte{}
  fmt.Printf("Result: %x\n", hasher.Sum(b))
  fmt.Printf("Result: %d\n", hasher.Sum(b))
  hasher.Reset()
  data := []byte("We shall overcome!")
  n, err := hasher.Write(data)
  if n!=len(data) || err!=nil {
    log.Printf("Hash write error: %v / %v", n, err)
  }
  checksum := hasher.Sum(b)
  fmt.Printf("Result: %x\n", checksum)
}
```

> 未來常用到的時候再補充

## Error-Handling and Testing

> go makes a distinction between critical and non-critical errors:
> 
> non-critical errors are returned as normal return values,
> 
> whereas for critical errors, the panic-recover mechanism is used.

Always assign an error to a variable within a compound if-statement; this makes for clearer code.

如果要停止執行程式的話，可以用 `os.Exit(1)`

透過 `errors.New("")` 建立新的 error-type
```go
err := errors.New("math - square root of negative number")
```

Convention
- Error type 以 `Error` 為 suffix (如: json.SyntaxError)
- Error variable 以 `err` 或 `Err` 為 prefix (如: syscall.Errno)

`fmt.Errorf()` 也是做成一個 error object

run-time panic
- 會搭配一個 runtime.Error 的 value
- program 會 crash 並提示 runtime.Error 的 message
- 這個 value 有 `RuntimeError()` 的方法

`defer` 可以在 panic 之後繼續執行

因為 panic 而程式中止，其中止流程稱為 `panicking`

> standard library 如果有 method 是以 `Must` 為 prefix 的話，表示出錯會造成 panic

`recover()` 是用來停止 panicking 的，可以重新拿回掌控權
- 只用在 `defer` function
- recover() return nil

範例
```go
func protect(g func()) {
  defer func() {
    log.Println("done") // Println executes normally even if there is a panic
    if err := recover(); err != nil {
      log.Printf("run time panic: %v", err)
    }
  }()
  log.Println("start")
  g() // possible runtime-error
}
```

> log.Fatal 會在紀錄 log 後呼叫 os.Exit(1)，立刻終止程式
> 
> log.Panic 會在紀錄 log 後呼叫 panic

另一個範例
```go
func badCall() {
  panic("bad end")
}

func test() {
  defer func() {
    if e := recover(); e != nil {
      fmt.Printf("Panicking %s\r\n", e);
    }
  }()
  badCall()
  fmt.Printf("After bad call\r\n");
}

func main() {
  fmt.Printf("Calling test\r\n");
  test()
  fmt.Printf("Test completed\r\n");
}
```

一定要在自己的 package 內對 panic 做 recover
- 不要跨越 package 的界線
- 回傳的錯誤要是自己 package 包過的 error value

> 要透過 defer - recover() 回傳 error 的話
> 
> 那個 function 的回傳值要先給他名稱
> 
> 如: `func Parse(input string) (numbers []int, err error)`

範例
```go
func Parse(input string) (numbers []int, err error) {
  defer func() {
    if r := recover(); r != nil {
      var ok bool
      err, ok = r.(error)
      if !ok {
        err = fmt.Errorf("pkg: %v", r)
      }
    }()

    fields := strings.Fields(input)
    numbers = fields2numbers(fields)
    return
}
```

如果每個 function 都要自己實作 recover() 的話會很亂

可參考下列方式 透過 closure 的做法在原本的流程以外補上 defer - recover()

```go
func errorHandler(fn fType1) fType1 {
  return func(a type1, b type2) {
    defer func() {
      if e, ok := recover().(error); ok {
        log.Printf("run time panic: %v", err)
      }
    }()
    fn(a, b)
  }
}
```

> `os.StartProcess()` 和 `exec.Command()` 都可以呼叫 os 執行 command

範例，panic 發生的話重啟程式
```go
func ultraCrazyFunction() {
    prog := os.Args[0]
    e := recover()
    if e != nil {
      // In case of panic, try to restart the entire application:
      cmd := exec.Command(prog)

      err := cmd.Run()
      // If that fails, log the error
      if err != nil {
        log.Fatal(err.Error())
      }
    }
  } 

func main() {
  defer ultraCrazyFunction()
}
```

`_test.go` 檔案是不會被 compile 的，只有在 `go test` 的時候才會

- `func (t *T) Fail()`: 測試失敗，但繼續執行
- `func (t *T) FailNow()`: 測試失敗，停止此檔案繼續執行，跳至下個檔案執行
- `func (t *T) Log(args ...interface{})`: text 會被記錄在 error-log
- `func (t *T) Fatal(args ...interface{})`: 功能是 `Log()` 加上 `FailNow()

> 測試 benchmark 的話是 `BenchmarkFunction` 為名稱
> 
> 並使用 *testing.B 參數，呼叫 `go test –test.bench=.*` 執行
 
```go
func BenchmarkReverse(b *testing.B) {
  ...
}
```

`go test -v <package name>` 可以單測指定的 package

**Investigating Performance**

紀錄 go test 的一些數據

輸出成檔案

```shell
go test -cpuprofile cpu.prof -memprofile mem.prof -bench 
```

透過 pprof 紀錄資訊

範例
```go
var cpuprofile = flag.String("cpuprofile", "", "write cpu profile to file")

func main() {
  flag.Parse()
  if *cpuprofile != "" {
    f, err := os.Create(*cpuprofile)
    if err != nil {
      log.Fatal(err)
    }
    defer f.close()
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
  }
  ...
}
```
執行程式後 會生成 對應的檔案
```shell
rogexec -cpuprofile=progexec.prof
```

接著就可以用 go tool 來查看
```shell
go tool pprof progexec.prof
```

其他類似的 go tool 還有
- top
  - 顯示 10 個 loading 最重的 function
- web or web funcname
  - 需安裝 `graphviz` 工具才可以看
  - 把 profile data 畫成 SVG 圖
  - 顯示 function 呼叫的流程圖
- list funcname or weblist funcname
  - 可以看出來哪段 code 最吃效能
  - 如果 `runtime.allocgc` 很高的話，最好是檢查一下

```go
var memprofile = flag.String("memprofile", "", "write memory profile to this file")
...

CallToFunctionWhichAllocatesLotsOfMemory()
if *memprofile != "" {
  f, err := os.Create(*memprofile)
  if err != nil {
    log.Fatal(err)
  }
  pprof.WriteHeapProfile(f)
  f.Close()
  return
}
```
指令 可以生成上面程式的 profile 檔案
```shell
progexec -memprofile=progexec.mprof
```
再用 go tool 工具看
```shell
go tool pprof progexec.mprof
```

## Goroutines and Channels

goroutine 執行的時候會使用 segmented stack 動態成長

stack management 是自動的

任何數量的 goroutine 都可以被更少數量的 thread 執行

```go
go func(){
}()
```

可以開始跑 function，在相同的 address，各自有自己的 stack

`init()` 時是可以運行 goroutine 的

使用 `GOMAXPROCS` 來提高 concurrent 的效能
- 告訴 runtime 有多少個 goroutine 可以平行的執行

```go
runtime.GOMAXPROCS(numCores)
```

goroutine 彼此間是獨立的運行單位

範例，使用 `sync.WaitGroup`
```go
func HeavyFunction1(wg *sync.WaitGroup) {
  defer wg.Done()
  // Do a lot of stuff
}

func HeavyFunction2(wg *sync.WaitGroup) {
  defer wg.Done()
  // Do a lot of stuff
}

func main() {
  wg := new(sync.WaitGroup)
  wg.Add(2)
  go HeavyFunction1(wg)
  go HeavyFunction2(wg)
  wg.Wait()
  fmt.Printf("All Finished!")
}
```

未初始化的 channel value 是 `nil`

```go
var identifier chan datatype

// 用 make 初始化
var ch1 chan string
ch1 = make(chan string)
// 或
ch1 := make(chan string)
// others
// channel of channels of int
chanOfChans := make(chan chan int)
// channel of functions
funcChan := make(chan func())
```

channel 事實上是一個 typed message queue，是 First In First Out 的結構

channel 是 reference type，使用 make 來配置記憶體位址

channel 是 first-class object，可以被存在變數，也可以傳給 function

甚至是把自己傳給 channel

channel 用完的話要關閉 `close(ch)`

傳送資料到 channel，`ch <- int1``

從 channel 拉資料出來 `int2 := <- ch`

命名習慣上會用 `ch` 或 包含 `chan`

> The channel `send` and `receive` operations are atomic which means they always complete without interruption.

因為可能有時間延遲，不要用 print 的方式來辨識 goroutine 的執行先後

By default, communication 是同步的，而且是 unbuffered
- 即 send 操作要等到 receiver 接收才算完成
- buffered channel 的溝通是 synchronous 的

deadlock 是指當 unbuffered channel 的 sender 和 receiver 同時等待彼此 

asynchronous channels 用 buffered channel
- 除非 buffer 滿了(才會造成 block)，不然 sender 可以一直 send 東西進 channel
- 除非 buffer 空了(才會造成 block)，不然 receiver 可以一直從 channel receive 東西
- `cap()` 可以回傳 channel 的  capacity
```go
buf := 100
ch1 := make(chan string, buf)
// buf 是指這個 channel 可以放多少 element 進去
```

> 設計時都先考慮 unbuffered channel，做不到或有問題時再改用 buffered channel

**semaphore pattern (旗號模式)**

當 concurrent goroutine 做完事情以後，發訊號進 channel 通知 main goroutine 他做完 (ready) 了
- 常見做法是先 block 住 main program，可把 `select{}` 放在 main 的後段

範例
```go
type Empty interface {}
var empty Empty
...
data := make([]float64, N)
res := make([]float64, N)
sem := make(chan Empty, N) // semaphore
...
for i, xi := range data {
  go func (i int, xi float64) {
    res[i] = doSomething(i,xi)
    sem <- empty
  } (i, xi)
}
// wait for goroutines to finish
for i := 0; i < N; i++ { <-sem }
```
要注意把 index (參數) 傳進 goroutine 的時候要用 copy

不然等到 goroutine 執行的時候如果被改掉，值就會不正確

透過 buffered channel 來做到 mutex
- buffered channel 的 capacity 是我們要 synchronize 的資源數量
- length (當前 buffered channel 的 item 數量) 是當前的資源使用情況
- capacity 扣掉 length 就是 channel 的 free resources 數量

範例
```go
type Empty interface {}
type semaphore chan Empty

sem = make(semaphore, N)

// acquire n resources
func (s semaphore) P(n int) {
    e := new(Empty)
    for i := 0; i < n; i++ {
        s <- e
    }
}
// release n resources
func (s semaphore) V(n int) {
    for i := 0; i < n; i++ {
        <-s
    }
}

// 實作 mutex
/* mutexes */
func (s semaphore) Lock() {
    s.P(1)
}
func (s semaphore) Unlock() {
    s.V(1)
}
/* signal-wait */
func (s semaphore) Wait(n int) {
    s.P(n)
}
func (s semaphore) Signal() {
    s.V(n)
}
```

**Channel Factory and Producer-Consumer Pattern**

不傳 channel 進 goroutine

讓 function 建立 channel 並回傳 (像 factory 來建立)

範例
```go
func main() {
    suck(pump())
    time.Sleep(1e9)
}

func pump() chan int {
    ch := make(chan int)
    go func() {
        for i := 0; ; i++ {
        	ch <- i
        }
    }()
    return ch
}

func suck(ch chan int) {
    go func() {
        for v := range ch {
            fmt.Println(v)
        }
    }()
}
```

用 for-range 來接收 channel 的資料

直到 channel close 以前都會一直處於讀取狀態 (沒收到資料的話會 block 住)

會透過另一個 goroutine 來傳送資料 並在結束時 close channel

```go
for v := range ch {
  fmt.Printf("The value is %v\n",v)
}
```

上面的範例可以整理成 Producer-Consumer pattern
- producer 傳出 channel 給 consumer 接來用

```go
for {
  Consume(Produce())
}
```

**Channel 的方向性**

```go
var send_only chan<- int // data can only be sent (written) to the channel
var recv_only <-chan int // data can only be received (read) from the channel
```

- `<-chan T` 是唯讀的 channel，無法被 close，因為 close 是給 sender goroutine 用的通知結束訊號，即不再傳送資料

```go
var c = make(chan int) // bidirectional
go source(c)
go sink(c)
func source(ch chan<- int) {
  for { ch <- 1 } // sending data to ch channel
}
func sink(ch <-chan int) {
  for { <-ch } // receiving data from ch channel
}
```

**Channel iterator pattern**

```go
func (c *container) Iter () <-chan items {
  ch := make(chan item)

  go func () {
    for i := 0; i < c.Len(); i++ { // or use a for-range loop
      ch <- c.items[i]
    }
  } ()
  return ch
}

// 透過上述的包裝，變成可以直接用 range 取值
for x := range container.Iter() { ... }
```

> 缺點: 如果程式異常終止的話，goroutine 不會被 GC 回收

**Pipe and filter pattern**

從一個 input channel 接收資料處例後再透過 output channel 送出資料

```go
sendChan := make(chan int)
receiveChan := make(chan string)
go processChannel(sendChan, receiveChan)
func processChannel(in <-chan int, out chan<- string) {
 for inValue := range in {
   result:= ... // processing inValue
   out <- result
 }
}
```

通道不總是需要被關閉，必要時才關，像是當 receiver 必須被告知已不需接收資料的時候

只有 sender 應該關閉通道，永遠不會是 receiver 來關

若通道已關閉
- 傳送資料進去會 panic
- 關閉已關閉通道會 panic

```go
// 好的習慣 用 defer 關閉
ch := make(chan float64)
defer close(ch)
```

```go
v, ok := <-ch // ok is true if v received a value

// 表示如果對已關閉 channel 拿資料會拿到 zero value, false

// 合併使用
if v, ok := <-ch; ok {
...
}

// 如果是在回圈內的話，可以使用 break 跳出
v, ok := <-ch
if !ok {
break
}
// process(v)
```

> 接收寫成 for-loop 比較好，因為會自動偵測 channel 是否被關閉

```go
for input := range ch {
  Process(input)
}
```

透過 `select` 搭配 `switch` 可以做到等待多個 go routine

稱為 `comminications switch`

**注意: 此種方法不能使用 fallthrough**

select 結束時機點
- 碰到 break
- 碰到 return
- 如果所有 case 都 block，就會一直 block 中
- 如果 default 以外的都 block，則執行 default
- 如果同時多個 case 可以取值，則隨機取

```go
select {
  case u:= <- ch1:
    ...
  case v:= <- ch2:
    ...
    ...
  default: // no value ready to be received
    ...
}
```

在 select 內使用 send operation 配上 `default` 的話

可以確保 send 會是 non-blocking 的

select 實作了一種 listener pattern，通常用在 infinite loop

當條件達到時才透過 `break` 做結束

> close 的 channel 可以隨意讀值 (空值)

**Nulling idiom**

```go
func iter(b int, c chan int) { 
  for i := 0; i < b; i++ {
    c <- i // put integers from 0 to n-1 on channel c
  }
  close(c) 
}

func main() {
  n, _ := strconv.Atoi(os.Args[1]) // line arg. converted to integer
  a := make(chan int)
  b := make(chan int)
  go iter(n, a)
  go iter(n, b)
  for a != nil || b != nil {
    select {
      case x, ok := <- a: // takes int value from channel a
        if ok {
     go     fmt.Println(x)
        } else { // if channel a is closed
          a = nil
        }
      case y, ok := <- b: // takes int value from channel b
        if ok {
          fmt.Println(y)
        } else { // if channel b is closed
          b = nil
        }
    }
  }
}
```

**Server backend pattern**

```go
// Backend goroutine.
func backend() {
  for {
    select {
      case cmd := <-ch1:
        // Handle ...
      case cmd := <-ch2:
        ...
      case cmd := <-chStop:
        // stop server
    }
  }
}
```

```go
func backend() {
  for req := range chRequest {
    switch req.Subject() {
      case A1: // Handle case ...
      case A2: // Handle case ...
      default:
        // Handle illegal request ...
        // ...
    }
  }
}
```

`time.Ticker` 會一直傳送時間給 `C`

呼叫 `Stop()` 會停止，通常用在 defer

```go
type Ticker struct {
  C <-chan Time // the channel on which the ticks are delivered.
  // contains filtered or unexported fields
  ...
}

// 使用情況
ticker := time.NewTicker(updateInterval)
defer ticker.Stop()...
select {
    case u:= <- ch1:    ...  
    case v:= <- ch2:    ...
    case <- ticker.C:   logState(status) // e.g. call some logging function logState
    default: // no value ready to be received    ...
}

// 另一個範例 透過 time.Tick 來做時間限制管理
import "time"
rate_per_sec := 10
var dur Duration = 1e9 / rate_per_sec
chRate := time.Tick(dur) // a tick every 1/10th of a second
for req := range requests {
	 <- chRate // rate limit our Service.Method RPC calls
	 go client.Call("Service.Method", req, ...)
}
```

範例
```go
func main() {
  tick := time.Tick(1e8)
  boom := time.After(5e8)
  for {
    select {
      case <-tick:
        fmt.Println("tick.")
      case <-boom:
        fmt.Println("BOOM!")
        return
      default:
        fmt.Println(" .")
        time.Sleep(5e7)
    }
  }
}
```
- `Tick()` 是固定 period 可以收到一次資料
- `After()` 是會在多久時間後收到唯一一次資料

**Simple Timeout Pattern**

版本 1

```go
timeout := make(chan bool, 1)
go func() {
  time.Sleep(1e9) // one second
  timeout <- true
}()

select {
    case <-ch:  // a read from ch has occurred
    case <-timeout:  // the read from ch has timed out
       break
}
```

版本 2

```go
ch := make(chan error, 1)
go func() {
  ch <- client.Call("Service.Method", args, &reply)
}()

select {
  case resp := <-ch: // use resp and reply
  case <-time.After(timeoutNs): // call timed out
    break
}
```

> buffer size 1 是必要的，用來避免 deadlock 並確保 timeout channel 可被 GC

版本 3 (範例: 只要最快跑完的那筆 query)
```go
func Query(conns []Conn, query string) Result {
  ch := make(chan Result, 1)
  for _, conn := range conns {
    go func(c Conn) {
      select {
        case ch <- c.DoQuery(query):
        default:
      }
    }(conn)
  }
  return <- ch
}
```

**Recovering with goroutines**

goroutine 內部自己可以做 defer recover

這樣掛掉的話不會影響其他 go routine

```go
func server(workChan <-chan *Work) {
  for work := range workChan {
    go safelyDo(work) // start the goroutine for that work
  }
}

func safelyDo(work *Work) {
  defer func() {
   if err := recover(); err != nil {
     log.Printf("work failed with %s in %v:", err, work)
     // the failing goroutine is stopped
   }
 }()
 do(work)
}
```

`recovver()` 如果不是放在 defer 的話，就只會回傳 `nil`

**Tasks and Worker Processes**

假設有多個 task 要執行，每個 task 由一個 worker 處理

```go
type Task struct {
  // some state
}
```

第一種方式: 透過 shared memory 來做 synchronize

tasks pool 是 shared memory. 為了 sync 工作且避免 race condition，要加上 `Mutex` lock

```go
type Pool struct {
  Mu sync.Mutex
  Tasks []Task
}

func Worker(pool *Pool) {
    for {
        pool.Mu.Lock()
        // begin critical section:
        task := pool.Tasks[0] // take the first task
        pool.Tasks = pool.Tasks[1:] // update the pool of tasks
        // end critical section
        pool.Mu.Unlock()
        process(task)
    }
}
```

一個 worker 鎖住 pool，從 pool 拿出第一個 task，再解鎖 pool，執行 task

lock 確保一次只有一個 worker 可以存取 pool

task 被指派給唯一一個 process

工作一多的話會頻繁的 lock - unlock，會降低效能

第二種方式: 使用 channel

使用 channels of `Tasks` 來做 synchronize
- 一個 pending channel 接收 requested tasks
- 一個 done channel 接收完成的 tasks (結果)

```go
func main() {
  pending, done := make(chan *Task), make(chan *Task)
  go sendWork(pending) // put tasks to do on the channel
  for i := 0; i < N; i++ { // start N goroutines to do work
    go Worker(pending, done)
  }
  consumeWork(done) // continue with the processed tasks
}

func Worker(in, out chan *Task) {
    for {
        t := <-in
        process(t)
        out <- t
    }
}
```

如果 tasks 增加了，就增加 worker 數量

從 channel 讀取和傳送資料是 atomic 操作

我們無法預期哪個 task 會被哪個 process 處理

**sync.Mutex vs Channel**

使用 `sync.Mutex` 當
- cache information 在 shared data structure 的時候
- holding state information (如: 執行中程序的 context 或 status)

使用 channels 當
- 通訊 asynchronous results
- 分散 units of work
- 傳遞 data 的 ownership

> 如果發現使用 locking 時的 rule 太複雜，可以考慮使用 channel 來做簡化

**實作 Lazy Generator**

generator 是一個會依序回傳下一個值的 function
```go
generateInteger() => 0
generateInteger() => 1
generateInteger() => 2
....
```

也稱作 Lazy Evaluation，表示只有在需要的時候計算，節省資源

範例
```go
var resume chan int
func integers() chan int {
  yield := make (chan int)
  count := 0
  go func () {
    for {
      yield <- count
      count++
    }
  } ()
  return yield
}

func generateInteger() int {
  return <-resume
}

func main() {
  resume = integers()
  fmt.Println(generateInteger()) //=> 0
  fmt.Println(generateInteger()) //=> 1
  fmt.Println(generateInteger()) //=> 2
}
```

很難的範例 (General Lazy Evaluation)
```go
type Any interface{}
type EvalFunc func(Any) (Any, Any)

func main() {
    evenFunc := func(state Any) (Any, Any) {
        os := state.(int)
        ns := os + 2
        return os, ns
    }

    even := BuildLazyIntEvaluator(evenFunc, 0)
    for i := 0; i < 10; i++ {
        fmt.Printf("%vth even: %v\n", i, even())
    }
}

func BuildLazyEvaluator(evalFunc EvalFunc, initState Any) func() Any {
    retValChan := make(chan Any)
    loopFunc := func() {
        var actState Any = initState
        var retVal Any
        for {
            retVal, actState = evalFunc(actState)
            retValChan <- retVal
        }
    }
    retFunc := func() Any {
        return <-retValChan
    }
    go loopFunc()
    return retFunc
}

func BuildLazyIntEvaluator(evalFunc EvalFunc, initState Any) func() int {
    ef := BuildLazyEvaluator(evalFunc, initState)
    return func() int {
        return ef().(int)
    }
}
```

> functions are values in Go, and so are closures.

**Typical client-server pattern**

```go
type Request struct {
a, b int
    replyc chan int // reply channel inside the Request
}

type binOp func(a, b int) int
func run(op binOp, req *Request) {
    req.replyc <- op(req.a, req.b)
}

func server(op binOp, service chan *Request) {
    for {
        req := <-service // requests arrive here
        
        // start goroutine for request:
        go run(op, req) // don't wait for op
    }
}

func startServer(op binOp) chan *Request {
    reqChan := make(chan *Request)
    go server(op, reqChan)
    return reqChan
}

func main() {
    adder := startServer(func(a, b int) int { return a + b })
    const N = 100
    var reqs [N]Request
    for i := 0; i < N; i++ {
        req := &reqs[i]
        req.a = i
        req.b = i + N
        req.replyc = make(chan int)
        adder <- req
    }
    // checks:
    for i := N - 1; i >= 0; i-- { // doesn't matter what order
        if <-reqs[i].replyc != N+2*i {
            fmt.Println("fail at", i)
        } else {
            fmt.Println("Request ", i, " is ok!")
        }
    }
    fmt.Println("done")
}
```

**Shutting down the server**

```go
func startServer(op binOp) (service chan *Request, quit chan bool) {
  service = make(chan *Request)
  quit = make(chan bool)
  go server(op, service, quit)
  return service, quit
}

func server(op binOp, service chan *request, quit chan bool) {
    for {
        select {
        case req := <-service:
            go run(op, req)
        case <-quit:
            return
        }
    }
}
```

典型的 sequential pipelining algorithm

```go
func SerialProcessData (in <- chan *Data, out chan <- *Data) {
  for data := range in {
    tmpA := PreprocessData(data)
    tmpB := ProcessStepA(tmpA)
    tmpC := ProcessStepB(tmpB)
    out <- PostProcessData(tmpC)
  }
}
```

改良
 
```go
func ParallelProcessData (in <-chan *Data, out chan <- *Data) {
  // make channels:
  preOut := make(chan *Data, 100)
  stepAOut := make(chan *Data, 100)
  stepBOut := make(chan *Data, 100)
  stepCOut := make(chan *Data, 100)
  // start parallel computations:
  go PreprocessData(in, preOut)
  go ProcessStepA(preOut, stepAOut)
  go ProcessStepB(stepAOut, stepBOut)
  go ProcessStepC(stepBOut, stepCOut)
  go PostProcessData(stepCOut, out)
}
```

## Networking, Templating and Web-Applications

TCP server 範例
```go
func main() {
  fmt.Println("Starting the server ...")
  // create listener:
  listener, err := net.Listen("tcp", "0.0.0.0:3001")
  if err != nil {
    fmt.Println("Error listening", err.Error())
    return // terminate program
  }
  // listen and accept connections from clients:
  for {
    conn, err := listener.Accept()
    if err != nil {
      return // terminate program
    }
    go doServerStuff(conn)
  }
}

func doServerStuff(conn net.Conn) {
  for {
    buf := make([]byte, 512)
    _, err := conn.Read(buf)
    if err != nil {
      fmt.Println("Error reading", err.Error())
      return // terminate program
    }
    fmt.Printf("Received data: %v", string(buf))
  }
}
```

`net.Listener` 是 server 的基本函式，用來監聽並接受 request
- 上面的範例是 IP-address 0.0.0.0 在 port 3001 透過 TCP-protocol

使用無窮迴圈搭配 `listener.Accept()`

client side 的 request 會建立出 `net.Conn` 的變數

server 要先啟動，這樣 client 才可以透過 `net.Diall()` 去建立連接 `conn` 變數
- Conn 是 interface，用來收發資料，所以 IPv4 / IPv6 / TCP / UDP 都可以共用 interface

透過 `conn.Write()` 把資料寫進去做傳輸

```go
conn, err:= net.Dial("tcp", "192.0.32.10:80") // tcp ipv4
checkConnection(conn, err)
conn, err = net.Dial("udp", "192.0.32.10:80") // udp
checkConnection(conn, err)
conn, err = net.Dial("tcp", "[2620:0:2d0:200::10]:80") // tcp ipv6
checkConnection(conn, err)
```

socket 的範例
```go
func main() {
  var (
    host = "www.apache.org"
    port = "80"
    remote = host + ":" + port
    msg string = "GET / \n"
    data = make([]uint8, 4096)
    read = true
    count = 0
  )
  // create the socket
  con, err := net.Dial("tcp", remote)
  // send our message. an HTTP GET request in this case
  io.WriteString(con, msg)
  // read the response from the webserver
  for read {
    count, err = con.Read(data)
    read = (err == nil)
    fmt.Printf(string(data[0:count]))
  }
  con.Close()
}
```

`net.ResolveTCPAddr` function 會回傳 `*net.TCPListener`

`net.Error`
```go
type Error interface {
  Timeout() bool // Is the error a timeout?
  Temporary() bool // Is the error temporary?
  ...
}
```

> HTTP 是 higher-level protocol than TCP

web-address 透過 `http.URL` 表達，其中包含了 `Path`

client-request 透過 `http.Request` 表示

req 如果是 post form 的話，可以用 `req.FormValue("val1")` 來取得值
- val1 是欄位名稱
- 適用於 https://xxx.xxx?var1=value
- 或是先呼叫 `req.ParseForm()`，再呼叫 `req.Form["var1"]`
- `Form` 其實是 `map[string][]string`
```go
var1, found := request.Form["var1"]
```

基本範例
```go
func HelloServer(w http.ResponseWriter, req *http.Request) {
  fmt.Println("Inside HelloServer handler")
  fmt.Fprint(w, "Hello, " + req.URL.Path[1:])
}

func main() {
  http.HandleFunc("/",HelloServer)
  err := http.ListenAndServe("0.0.0.0:3000", nil)
  if err != nil {
    log.Fatal("ListenAndServe: ", err.Error())
  }
}
```

只爬 head 的小爬蟲範例
```go
var urls = []string{
  "http://www.google.com/",
  "http://golang.org/",
  "http://blog.golang.org/",
}

func main() {
  // Executes an HTTP HEAD request for all URL's
  // and returns the HTTP status string or an error string.
  for _, url := range urls {
    resp, err := http.Head(url)
    if err != nil {
    fmt.Println("Error:", url, err)
  }
  fmt.Println(url, ": ", resp.Status)
  }
}
```

爬網頁的範例
```go
func main() {
  res, err := http.Get("http://www.google.com")
  CheckError(err)
  data, err := ioutil.ReadAll(res.Body)
  CheckError(err)
  fmt.Printf("Got: %q", string(data))
}

func CheckError(err error) {
  if err != nil {
    log.Fatalf("Get: %v", err)
  }
}
```

其他 `http` package 有用的 function
- `http.Redirect(w ResponseWriter, r *Request, url string, code int)`
  - 重新導向
- `http.NotFound(w ResponseWriter, r *Request)`
  - 回應 404 not found error
- `http.Error(w ResponseWriter, error string, code int)`
  - 回應特殊的訊息和 HTTP code
- A useful field of a http.Request object req is: req.Method
  - 取得是 GET 或是 POST

透過 `w.Header().Set("Content-Type", "text/html")` 可以設定 header

`http.DetectContentType([]byte(form))` 可以取得 content-type

透過 middleware 來做到 panic recover
```go
func logPanics(function HandleFnc) HandleFnc {
  return func(writer http.ResponseWriter, request *http.Request) {
    defer func() {
      if x := recover(); x != nil {
        log.Printf("[%v] caught panic: %v", request.RemoteAddr, x)
      }
    }()
  function(writer, request)
 }
}

http.HandleFunc("/test1", logPanics(SimpleServer))
http.HandleFunc("/test2", logPanics(FormServer))
```

驗證 url 的有效性
- `var validPath = regexp.MustCompile("^/(edit|save|view)/([a-zA-Z0-9]+)$")`

`template` package 的 embedded 寫法
```go
<h1>{{.Title |html}}</h1>
<p>[<a href="/edit/{{.Title |html}}">edit</a>]</p>
<div>{{printf "%s" .Body |html}}</div>
```

`template.Must(template.ParseFiles("edit.html", "view.html"))` 把資料轉換至 `*template.Template`
- 為了 efficiency 的原因，做一次就可以了

建立 page，透過 template 和 struct
```go
templates.ExecuteTemplate(w, tmpl+".html", p)
```

> 關於 template 就不多做敘述，需要用到實在另外找資源
> 
> 主要都是在 template 檔案上面的語法操作
> 
> 關於 rpc 和 smtp 也不多做敘述

## Regular Mistakes and Suggestions

- 不要忘記在終止 buffered writing 的時候使用 `Flush()`
- 不要用 global variable 或 shared memory，會在 concurrency 的時候 unsafe
- 使用 JSON 的時候，注意欄位大小寫 -> 是否 exported
- println 只用在 debug

Best practices
- 正確的初始化 slice of maps
- type assertion 總是使用 `comma, ok`
- 使用 factory 來 建立並初始化 types
- 在會更改 struct 本身時 method 才用 pointer receiver，不然都用 value receiver
- HTML output 生成時總是用 `html/tempate`

常見的問題
- 字串連接不要用 `+`，因為字串在 golang 是 immutable，所以會一直重新 reallocate 和 copy memory
  - 建議使用 `bytes.Buffer` 來做字串的串接 (特別是當串接數量大於 15 後)
```go
var b bytes.Buffer
...
for condition {
  b.WriteString(str) // appends string str to the buffer
}
return b.String()
```

- defer 只有在 return function 的時候會執行，不是在 for loop 結束或其他有限制的 scope
  - 即不要 loop 內使用 defer
- slice 本身已經是 reference，不要再傳他的 reference 到 function 內
  - 當使用 slice parameter 的時候，不需要再 dereference
- interface 已經是 reference type，不要再用他的 reference
  - never use a pointer to an interface type; this is already a pointer
- 傳參數到 function 時，能用 value 就少用 pointer
  - 傳 value 是放在 stack
  - 傳 pointer 是放在 heap，會有額外的記憶體配置

用到 defer 的情境
- 關閉檔案串流
- 解鎖 mutex
- 關閉 channel
- recover from panic
- 停止 time ticker
- release process
- 停止 CPU profiling and flushing the info
- goroutine signaling a waitGroup

## Performance Advices
字串內的更改
- 因為 string 是 immutable，所以要另外建立新的 string

```go
str := "hello"
c := []rune(s)
c[0] = 'c'
s2 := string(c) // s2 == "cello"
```

取得 substring
```go
substr := str[n:m]
```

對 string 做 loop

```go
// gives only the bytes:
for i:=0; i < len(str); i++ {
  ... = str[i]
}
// gives the Unicode characters:
for ix, ch := range str {
  ...
}
```

找尋 string 的 number of bytes 和字元

```go
// 取得 number of bytes
len(str)

// 取得 number of characters
utf8.RuneCountInString(str)
// 或
len([]int(str))
```

字串串接

```go
// 最快的方法
// with a bytes.Buffer 
var buffer bytes.Buffer
var s string
buffer.WriteString(s)
fmt.Print(buffer.String(), "\n")

// 或
Strings.Join() // using Join function
str1 += str2 // using += operator
```

array 和 slice 的操作
```go
// 建立 array
arr1 := new([len]type)
// 建立 slice
slice1 := make([]type, len)
// 初始化 array
arr1 := [...]type{i1, i2, i3, i4, i5}
arrKeyValue := [len]type{i1: val1, i2: val2}
// 初始化 slice
var slice1 []type = arr1[start:end]
// 切掉最後一個元素
line = line[:len(line)-1]
// loop
for i:=0; i < len(arr); i++ {
... = arr[i]
}

for ix, value := range arr {
...
}
```

**Structs, Interfaces and Maps**

基本語法
```go
// Creation
type struct1 struct {
field1 type1
field2 type2
...
}

ms := new(struct1)

// initialization
ms := &struct1{10, 15.5, "Chris"}
// or
ms := Newstruct1{10, 15.5, "Chris"}

func Newstruct1(n int, f float32, name string) *struct1 {
    return &struct1{n, f, name}
}
```

```go
// testing if a value implements an inerface
if v, ok := v.(Stringer); ok { // test if v implements Stringer
    fmt.Printf("implements String(): %s\n", v.String());
}

// type classifier
func classifier(items ...interface{}) {
    for i, x := range items {
        switch x.(type) {
            case bool: fmt.Printf("param #%d is a bool\n", i)
            case float64: fmt.Printf("param #%d is a float64\n", i)
            case int, int64: fmt.Printf("param #%d is an int\n", i)
            case nil: fmt.Printf("param #%d is nil\n", i)
            case string: fmt.Printf("param #%d is a string\n", i)
            
            default: fmt.Printf("param #%d's type is unknown\n", i)
        }
    }
}
```

```go
// creation
map1 := make(map[keytype]valuetype)

// Initialization
map1 := map[string]int{"one": 1, "two": 2}

// loop
for key, value := range map1 {
...
}

// testing if a key value exists in a map
val1, isPresent = map1[key1]

// deleting a key in a map
delete(map1, key1)
```

```go
// recovering to stop a panic terminating sequence
func protect(g func()) {
    defer func() {
        log.Println("done") // Println executes normally even if there is a panic
        if x := recover(); x != nil {
            log.Printf("run time panic: %v", x)
        }
    }()
    log.Println("start")
    g()
}

// opening and reading a file
file, err := os.Open("input.dat")
if err!= nil {
    fmt.Printf("An error occurred on opening the inputfile\n" +
    "Does the file exist?\n" +
    "Have you got acces to it?\n")
    return
}
defer file.Close()
iReader := bufio.NewReader(file)
for {
    str, err := iReader.ReadString('\n')
    if err!= nil {
        return // error or EOF
    }
    fmt.Printf("The input was: %s", str)
}

// copying a file with a buffer
func cat(f *file.File) {
    const NBUF = 512
    var buf [NBUF]byte
    for {
        switch nr, er := f.Read(buf[:]); true {
            case nr < 0:
                fmt.Fprintf(os.Stderr, "cat: error reading from %s: %s\n", f.String(),
                er.String())
                os.Exit(1)
            case nr == 0: // EOF
                return
            case nr > 0:
                if nw, ew := file.Stdout.Write(buf[0:nr]); nw != nr {
                    fmt.Fprintf(os.Stderr, "cat: error writing from %s: %s\n", f.String(),
                    ew.String())
                }
        }
    }
}
```

> The amount of work done inside goroutines has to be much larger than the costs associated with creating goroutines and sending data back and forth between them.
> 
> Using buffered channels for performance: A buffered channel can easily double its throughput depending on the context, and the performance gain can be 10x or more. You can further try to optimize by adjusting the capacity of the channel.
> 
> Limiting the number of items in a channel and packing them in arrays: Channels become a bottleneck if you pass a lot of individual items through them. You can work around this by packing chunks of data into arrays and then unpacking on the other end. This can give a speed gain of a factor 10x.

```go
// creation
ch := make(chan type, buf)

// loop
for v := range ch {
// do something with v
}

// testing if a channel is closed
//read channel until it closes or error-condition
for {
    if input, open := <-ch; !open {
        break
    }
    fmt.Printf("%s ", input)
}

// Or, use the looping over a channel method where the detection is automatic.

// using a channel to let the main program wait
// semaphore pattern
ch := make(chan int) // Allocate a channel

// Start something in a goroutine; when it completes, signal on the channel
go func() {
    // doSomething
    ch <- 1 // Send a signal; value does not matter
}()
doSomethingElseForAWhile()
<-ch // Wait for goroutine to finish; discard sent value

// channel factory pattern
func pump() chan int {
    ch := make(chan int)
    go func() {
        for i := 0; ; i++ {
            ch <- i
        }
    }()
    return ch
}

// stopping a goroutine
runtime.Goexit()

// simple timeout pattern
timeout := make(chan bool, 1)
go func() {
    time.Sleep(1e9) // one second
    timeout <- true
}()
select {
    case <-ch:
    // a read from ch has occurred
    case <-timeout:
    // the read from ch has timed out
}

// using an in- and out-channel instead of locking
func Worker(in, out chan *Task) {
    for {
        t := <-in
        process(t)
        out <- t
    }
}
```

```go
// templating
// Make, parse and validate a template
var strTempl = template.Must(template.New("TName").Parse(strTemplateHTML))

// Use the html filter to escape HTML special characters, when used in a web context
{{html .}} or with a field FieldName {{ .FieldName |html }}

// Use template-caching
```

```go
// stopping a program in case of error
if err != nil {
    fmt.Printf("Program stopping with error %v", err)
    os.Exit(1)
}
// or
if err != nil {
    panic("ERROR occurred: " + err.Error())
}
```

**Performance best practices and advice**
- 如果可以的話，使用 `[]rune` 優於 strings
- 使用 slices 優於 arrays
- 使用 arrays 或 slices 優於 map 
- 不需要 index 的時候 loop 使用 `for range`，會比 loop 每個 element 還快
- 如果 array 的元素大部分是 0 或 nil 的話，使用 map 可以減少記憶體用量
- specify initial capacity for maps
- 定義 method 的時候，使用 pointer to a type (struct) 作為 receiver
- 使用 constants 或 flags 來提取常數
- use caching whenever possible when large amounts of memory are being allocated
- use template caching

> sparse: containing many 0 or nil-values

## Reference
[Golang Interview Questions](https://www.interviewbit.com/golang-interview-questions/)
