+++
title = 'Go 語言系列（五）：錯誤處理與併發'
date = 2026-02-24T05:00:00+08:00
draft = false
tags = ['Golang', 'Go', '錯誤處理', '併發', 'Goroutine', 'Channel', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '學習 Go 的錯誤處理模式與併發程式設計：error interface、defer/panic/recover、Goroutine、Channel 和 sync 套件'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第五篇，也是基礎篇的最後一篇。這篇要學的是 Go 最有特色的兩個主題：錯誤處理和併發。

## 錯誤處理

### Go 的錯誤哲學

Go 沒有 try/catch。它的哲學是：**錯誤是正常的程式流程，應該被明確處理，而不是被「拋出」和「捕捉」**。

Go 使用多值回傳來傳遞錯誤：

```go
result, err := doSomething()
if err != nil {
    // 處理錯誤
    return err
}
// 使用 result
```

這個模式你在 Go 程式碼中會看到無數次。

### error 介面

Go 的 error 是一個簡單的介面：

```go
type error interface {
    Error() string
}
```

任何實作了 `Error() string` 方法的型別都是 error。

### 建立錯誤

```go
import (
    "errors"
    "fmt"
)

// 方式一：errors.New
err := errors.New("something went wrong")

// 方式二：fmt.Errorf（可以格式化訊息）
name := "config.json"
err = fmt.Errorf("file not found: %s", name)
```

### 自訂 Error 型別

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}

func validateAge(age int) error {
    if age < 0 || age > 150 {
        return &ValidationError{
            Field:   "age",
            Message: "must be between 0 and 150",
        }
    }
    return nil
}
```

### 錯誤包裝（Error Wrapping）

Go 1.13 引入了錯誤包裝機制，讓你可以在傳遞錯誤時加入上下文：

```go
// 包裝錯誤：用 %w 動詞
func readConfig(path string) error {
    data, err := os.ReadFile(path)
    if err != nil {
        return fmt.Errorf("reading config %s: %w", path, err)
    }
    // ... 使用 data
    return nil
}

// 解開錯誤
err := readConfig("config.json")

// errors.Is：檢查錯誤鏈中是否包含特定錯誤
if errors.Is(err, os.ErrNotExist) {
    fmt.Println("Config file does not exist")
}

// errors.As：取出特定型別的錯誤
var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Println("Failed path:", pathErr.Path)
}
```

### 錯誤處理的最佳實踐

```go
// 好的做法：立即處理錯誤
user, err := findUser(id)
if err != nil {
    return fmt.Errorf("finding user %d: %w", id, err)
}

// 不好的做法：忽略錯誤
user, _ := findUser(id)  // 危險！
```

原則：

1. **永遠檢查錯誤**，除非你有充分理由忽略
2. **加入上下文**再傳遞錯誤（用 `fmt.Errorf` 和 `%w`）
3. **只在最終呼叫者記錄錯誤**，中間層只傳遞
4. **使用 sentinel errors**（如 `ErrNotFound`）表達已知的錯誤情境

## defer

`defer` 會在函式結束時執行指定的語句，無論是正常 return 還是 panic。

### 基本用法

```go
func readFile(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return err
    }
    defer f.Close()  // 函式結束時一定會關閉檔案

    // 讀取和處理檔案...
    return nil
}
```

### 多個 defer

多個 defer 按照 **LIFO（後進先出）** 的順序執行：

```go
func example() {
    defer fmt.Println("1")
    defer fmt.Println("2")
    defer fmt.Println("3")
}
// 輸出：3, 2, 1
```

### 常見用途

```go
// 關閉資源
defer file.Close()
defer db.Close()
defer resp.Body.Close()

// 解鎖
mu.Lock()
defer mu.Unlock()

// 計時
func timeIt(name string) func() {
    start := time.Now()
    return func() {
        fmt.Printf("%s took %v\n", name, time.Since(start))
    }
}

func doWork() {
    defer timeIt("doWork")()
    // ... 做一些事情
}
```

## panic 與 recover

### panic

`panic` 類似其他語言的 throw，會中斷正常執行流程：

```go
func mustParseInt(s string) int {
    n, err := strconv.Atoi(s)
    if err != nil {
        panic(fmt.Sprintf("cannot parse %q as int: %v", s, err))
    }
    return n
}
```

**何時使用 panic？**

- 程式遇到不可能復原的錯誤（初始化失敗、不可能的狀態）
- 違反了程式的不變量（invariant）
- 大多數情況下，**回傳 error 比 panic 更好**

### recover

`recover` 可以捕獲 panic，防止程式崩潰：

```go
func safeFunction() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()

    // 可能 panic 的程式碼
    panic("something bad happened")
}

func main() {
    safeFunction()
    fmt.Println("Program continues...")  // 這行會被執行
}
```

`recover` 只在 `defer` 函式中有效。在 Web 應用中，我們會用 recovery middleware 來防止單一請求的 panic 影響整個伺服器。

## Goroutine

Goroutine 是 Go 的輕量級執行緒。一個 Goroutine 只需要約 2KB 的記憶體（OS thread 通常需要 1MB+），可以輕鬆建立數十萬個。

### 啟動 Goroutine

```go
func sayHello(name string) {
    fmt.Printf("Hello, %s!\n", name)
}

func main() {
    go sayHello("Alice")  // 在新的 goroutine 中執行
    go sayHello("Bob")

    // 等一下，否則 main 結束時所有 goroutine 都會被終止
    time.Sleep(time.Second)
}
```

`go` 關鍵字就這麼簡單——在函式呼叫前加上 `go`，它就會在新的 goroutine 中非同步執行。

### 匿名函式 Goroutine

```go
go func() {
    fmt.Println("Running in a goroutine")
}()

go func(msg string) {
    fmt.Println(msg)
}("Hello from goroutine")
```

## Channel

Channel 是 goroutine 之間的通訊管道。Go 的併發哲學是：

> **不要透過共享記憶體來通訊，而是透過通訊來共享記憶體。**

### 建立和使用

```go
// 建立 channel
ch := make(chan string)

// 在 goroutine 中送出訊息
go func() {
    ch <- "Hello"  // 送出
}()

// 在主 goroutine 中接收
msg := <-ch  // 接收（會阻塞直到有訊息）
fmt.Println(msg)  // Hello
```

### 有緩衝的 Channel

```go
// 無緩衝：送出和接收必須同步（都準備好才能交換）
ch1 := make(chan int)

// 有緩衝：可以暫存指定數量的值
ch2 := make(chan int, 3)
ch2 <- 1  // 不阻塞
ch2 <- 2  // 不阻塞
ch2 <- 3  // 不阻塞
// ch2 <- 4  // 阻塞！緩衝已滿
```

### 關閉 Channel

```go
ch := make(chan int)

go func() {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)  // 送完了，關閉 channel
}()

// 用 range 接收直到 channel 關閉
for v := range ch {
    fmt.Println(v)
}
```

### 單向 Channel

```go
// 只能送出的 channel
func producer(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)
}

// 只能接收的 channel
func consumer(ch <-chan int) {
    for v := range ch {
        fmt.Println(v)
    }
}

ch := make(chan int)
go producer(ch)
consumer(ch)
```

### select 語句

`select` 讓你同時等待多個 channel 操作：

```go
ch1 := make(chan string)
ch2 := make(chan string)

go func() {
    time.Sleep(1 * time.Second)
    ch1 <- "one"
}()

go func() {
    time.Sleep(2 * time.Second)
    ch2 <- "two"
}()

// 等待哪個先到
select {
case msg := <-ch1:
    fmt.Println("Received from ch1:", msg)
case msg := <-ch2:
    fmt.Println("Received from ch2:", msg)
case <-time.After(3 * time.Second):
    fmt.Println("Timeout!")
}
```

## sync 套件

### WaitGroup

`sync.WaitGroup` 用來等待一組 goroutine 完成：

```go
var wg sync.WaitGroup

urls := []string{
    "https://example.com",
    "https://example.org",
    "https://example.net",
}

for _, url := range urls {
    wg.Add(1)  // 計數器 +1
    go func(u string) {
        defer wg.Done()  // 計數器 -1
        fmt.Printf("Fetching %s\n", u)
        // ... 實際的 HTTP 請求
    }(url)
}

wg.Wait()  // 等待計數器歸零
fmt.Println("All done!")
```

### Mutex

`sync.Mutex` 用來保護共享資源：

```go
type SafeCounter struct {
    mu    sync.Mutex
    count int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

counter := &SafeCounter{}

var wg sync.WaitGroup
for i := 0; i < 1000; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        counter.Increment()
    }()
}
wg.Wait()

fmt.Println(counter.Value())  // 1000（沒有 race condition）
```

## 常見併發模式

### Worker Pool

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, j)
        time.Sleep(time.Second)  // 模擬工作
        results <- j * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)

    // 啟動 3 個 worker
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }

    // 送出 9 個 job
    for j := 1; j <= 9; j++ {
        jobs <- j
    }
    close(jobs)

    // 收集結果
    for r := 1; r <= 9; r++ {
        fmt.Println(<-results)
    }
}
```

### 使用 context 控制取消

```go
import "context"

func longRunningTask(ctx context.Context) error {
    for i := 0; i < 10; i++ {
        select {
        case <-ctx.Done():
            fmt.Println("Task cancelled")
            return ctx.Err()
        default:
            fmt.Printf("Working... step %d\n", i)
            time.Sleep(time.Second)
        }
    }
    return nil
}

func main() {
    // 建立一個 3 秒後自動取消的 context
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()

    err := longRunningTask(ctx)
    if err != nil {
        fmt.Println("Error:", err)
    }
}
```

`context` 在 Web 應用中非常重要——每個 HTTP 請求都帶有 context，用來處理超時和取消。

## 練習：並行網頁擷取器

```go
package main

import (
    "fmt"
    "net/http"
    "sync"
    "time"
)

type Result struct {
    URL      string
    Status   int
    Duration time.Duration
    Error    error
}

func fetch(url string) Result {
    start := time.Now()
    resp, err := http.Get(url)
    duration := time.Since(start)

    if err != nil {
        return Result{URL: url, Error: err, Duration: duration}
    }
    defer resp.Body.Close()

    return Result{
        URL:      url,
        Status:   resp.StatusCode,
        Duration: duration,
    }
}

func main() {
    urls := []string{
        "https://go.dev",
        "https://github.com",
        "https://example.com",
    }

    results := make([]Result, len(urls))
    var wg sync.WaitGroup

    start := time.Now()

    for i, url := range urls {
        wg.Add(1)
        go func(idx int, u string) {
            defer wg.Done()
            results[idx] = fetch(u)
        }(i, url)
    }

    wg.Wait()
    totalDuration := time.Since(start)

    fmt.Println("=== 結果 ===")
    for _, r := range results {
        if r.Error != nil {
            fmt.Printf("%-30s Error: %v (%v)\n", r.URL, r.Error, r.Duration)
        } else {
            fmt.Printf("%-30s Status: %d (%v)\n", r.URL, r.Status, r.Duration)
        }
    }
    fmt.Printf("\n總耗時: %v（並行執行）\n", totalDuration)
}
```

因為三個請求是並行執行的，總耗時會接近最慢的那個請求，而非三個請求的時間總和。

## 基礎篇總結

到這裡，基礎篇的 5 篇文章就結束了。讓我們回顧學到了什麼：

1. **為什麼學 Go**：設計哲學、適用場景
2. **環境設定**：安裝、IDE、Go Modules
3. **基礎語法**：變數、型別、流程控制、函式
4. **資料結構**：Slice、Map、Struct、Interface
5. **錯誤處理與併發**：error、Goroutine、Channel

有了這些基礎，你已經準備好進入實戰篇了！

## 下一步

從下一篇開始，我們進入**實戰篇**。第六篇將帶你用 Go 的標準庫建立第一個 HTTP Web Server，開始從基礎知識走向實際的 Web 應用開發。
