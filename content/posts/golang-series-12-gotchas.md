+++
title = 'Go 語言系列（十二）：常見陷阱與反模式'
date = 2026-02-24T12:00:00+08:00
draft = false
tags = ['Golang', 'Go', '陷阱', 'Gotchas', '反模式', '最佳實踐', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '整理 Go 開發中最常見的陷阱與反模式：Slice 共享、Closure 捕獲、nil interface、Map 併發、Goroutine 洩漏等，每個陷阱附帶錯誤與正確的對照程式碼'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第十二篇，也是本系列的最終篇。這篇整理 Go 開發中最常踩到的陷阱——即使是有經驗的 Go 開發者也可能中招。每個陷阱都附帶「踩坑範例」和「正確做法」，讓你一次看清問題在哪。

## Slice 陷阱

### 陷阱 1：Append 可能修改原始 Slice

當 slice 的 capacity 還有剩餘空間時，`append` 會直接修改底層 array，影響到共享同一個 array 的其他 slice。

```go
package main

import "fmt"

func main() {
    original := make([]int, 3, 5) // len=3, cap=5
    original[0], original[1], original[2] = 1, 2, 3

    // 截取一個子 slice
    sub := original[:2]

    // append 到 sub — cap 還有空間，不會分配新 array
    sub = append(sub, 99)

    fmt.Println("original:", original) // [1 2 99] ← 被改了！
    fmt.Println("sub:", sub)           // [1 2 99]
}
```

**正確做法：** 使用 full slice expression 限制 capacity

```go
// 用 original[:2:2] 限制 sub 的 cap = 2
sub := original[:2:2]

// 現在 append 會分配新 array，不影響 original
sub = append(sub, 99)
fmt.Println("original:", original) // [1 2 3] ← 沒被改
fmt.Println("sub:", sub)           // [1 2 99]
```

### 陷阱 2：大 Slice 截取小片段導致記憶體無法釋放

```go
// 錯誤：小 slice 持有大 array 的引用，GC 無法回收
func getFirstTen(data []byte) []byte {
    return data[:10] // 底層 array 整個被保留
}

// 正確：複製需要的資料到新 slice
func getFirstTen(data []byte) []byte {
    result := make([]byte, 10)
    copy(result, data[:10])
    return result // 原始的大 array 可以被 GC 回收
}
```

假設 `data` 是 1GB 的檔案內容，用截取的方式只取 10 bytes，那 1GB 的底層 array 都不會被回收。

## Closure 變數捕獲陷阱

### 陷阱 3：For Loop 中的 Closure 捕獲

這是 Go 最經典的陷阱之一：

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup

    // Go 1.21 及之前：所有 goroutine 都印出相同的值
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            fmt.Println(i) // 捕獲的是變數 i 本身，不是值
        }()
    }
    wg.Wait()
    // 可能輸出：5 5 5 5 5（全部都是迴圈結束後的值）
}
```

**原因：** 在 Go 1.21 及之前，for loop 的迴圈變數只有一份，所有 closure 都捕獲同一個變數的 reference。當 goroutine 真正執行時，迴圈已經結束，`i` 的值已經是 5。

**修正方式（Go 1.21 及之前）：**

```go
for i := 0; i < 5; i++ {
    wg.Add(1)
    i := i // 建立一個新的局部變數，shadow 迴圈變數
    go func() {
        defer wg.Done()
        fmt.Println(i)
    }()
}
```

**Go 1.22+ 的修正：** 從 Go 1.22 起，每次迭代的迴圈變數都是一個新的實例，這個陷阱已經被語言層面修正了。但如果你的專案需要支援舊版本，仍然要注意。

## Nil Interface 陷阱

### 陷阱 4：持有 Nil Pointer 的 Interface 不等於 Nil

```go
package main

import "fmt"

type MyError struct {
    Message string
}

func (e *MyError) Error() string {
    return e.Message
}

func doSomething() error {
    var err *MyError // nil pointer
    // ... 一些邏輯 ...
    return err // 回傳 nil pointer，但型別資訊被帶入 interface
}

func main() {
    err := doSomething()
    if err != nil {
        fmt.Println("got error!") // 會進來！即使 err 的值是 nil
    }
}
```

**原因：** Go 的 interface 內部是 `(type, value)` 的組合。`doSomething()` 回傳的 interface 是 `(*MyError, nil)`——它有 type 資訊，所以不等於 `nil`。只有 `(nil, nil)` 才等於 `nil`。

**正確做法：**

```go
func doSomething() error {
    var err *MyError
    // ... 一些邏輯 ...
    if err == nil {
        return nil // 直接回傳 nil，不帶型別資訊
    }
    return err
}
```

## Map 併發陷阱

### 陷阱 5：併發讀寫 Map 會 Panic

```go
package main

import "sync"

func main() {
    m := make(map[int]int)
    var wg sync.WaitGroup

    // 這會觸發 fatal error: concurrent map writes
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            m[n] = n // 併發寫入
        }(i)
    }
    wg.Wait()
}
```

Go 的 runtime 會偵測到 map 的併發寫入，直接觸發 `fatal error`（不是 panic，無法 recover）。

**正確做法：**

```go
// 方案一：sync.RWMutex
var mu sync.RWMutex
m := make(map[int]int)

// 寫入
mu.Lock()
m[key] = value
mu.Unlock()

// 讀取
mu.RLock()
v := m[key]
mu.RUnlock()

// 方案二：sync.Map（讀多寫少的場景更高效）
var sm sync.Map
sm.Store(key, value)
v, ok := sm.Load(key)
```

### 陷阱 6：Range Map 的順序不固定

```go
package main

import "fmt"

func main() {
    m := map[string]int{"a": 1, "b": 2, "c": 3}

    // 每次執行的輸出順序可能不同
    for k, v := range m {
        fmt.Printf("%s=%d ", k, v)
    }
    // 可能是 "b=2 c=3 a=1" 或 "a=1 c=3 b=2" 或其他順序
}
```

Go 刻意隨機化 map 的迭代順序，防止開發者依賴特定順序。如果需要固定順序，先取出 keys 排序。

## String 與 Byte 陷阱

### 陷阱 7：Range String 是按 Rune 迭代，不是 Byte

```go
package main

import "fmt"

func main() {
    s := "Hello 你好"

    // range 按 rune 迭代
    for i, r := range s {
        fmt.Printf("index=%d rune=%c bytes=%d\n", i, r, len(string(r)))
    }
    // index=0 rune=H bytes=1
    // index=1 rune=e bytes=1
    // ...
    // index=6 rune=你 bytes=3  ← index 跳了！
    // index=9 rune=好 bytes=3

    fmt.Println("len(s):", len(s))         // 12（byte 數）
    fmt.Println("runes:", len([]rune(s)))   // 8（字元數）
}
```

`len(string)` 回傳的是 **byte 數**，不是字元數。中文字在 UTF-8 中佔 3 個 bytes。

### 陷阱 8：String 與 []byte 轉換有效能開銷

```go
s := "hello"
b := []byte(s) // 分配新記憶體，複製資料
s2 := string(b) // 又分配新記憶體，複製資料
```

每次轉換都會分配新記憶體和複製。在高頻呼叫的路徑上要注意。如果只需要讀取，考慮使用 `strings.Builder` 或 `bytes.Buffer` 來減少分配。

## Goroutine 洩漏陷阱

### 陷阱 9：Channel 未關閉導致 Goroutine 洩漏

```go
// 錯誤：producer goroutine 永遠不會結束
func leakyProducer() <-chan int {
    ch := make(chan int)
    go func() {
        for i := 0; ; i++ {
            ch <- i // 如果沒人讀取，這裡永遠阻塞
        }
    }()
    return ch
}

// 正確：用 context 控制 goroutine 生命週期
func producer(ctx context.Context) <-chan int {
    ch := make(chan int)
    go func() {
        defer close(ch)
        for i := 0; ; i++ {
            select {
            case ch <- i:
            case <-ctx.Done():
                return // context 被取消時，goroutine 優雅退出
            }
        }
    }()
    return ch
}
```

**經驗法則：** 每個 goroutine 都要有明確的結束條件。啟動 goroutine 時，問自己：「它什麼時候會結束？」

### 陷阱 10：Select 缺少 Timeout

```go
// 錯誤：如果 ch 永遠沒有資料，goroutine 永遠卡住
select {
case v := <-ch:
    process(v)
}

// 正確：加上 timeout
select {
case v := <-ch:
    process(v)
case <-time.After(5 * time.Second):
    log.Println("timeout waiting for data")
    return
}
```

## Error 處理陷阱

### 陷阱 11：忽略 Error 回傳值

```go
// 錯誤：忽略 error
json.Unmarshal(data, &result)
os.Remove(filename)
fmt.Fprintf(w, "hello")

// 正確：處理每個 error
if err := json.Unmarshal(data, &result); err != nil {
    return fmt.Errorf("unmarshaling: %w", err)
}
```

Go 的 linter（如 `errcheck`）可以幫你抓到被忽略的 error。

### 陷阱 12：用 == 比較 Wrapped Error

```go
package main

import (
    "errors"
    "fmt"
    "io"
)

func readData() error {
    // 假設底層回傳 io.EOF
    return fmt.Errorf("reading failed: %w", io.EOF)
}

func main() {
    err := readData()

    // 錯誤：== 比較 wrapped error 會失敗
    if err == io.EOF {
        fmt.Println("got EOF") // 不會進來
    }

    // 正確：用 errors.Is 解開 wrap 鏈
    if errors.Is(err, io.EOF) {
        fmt.Println("got EOF (via errors.Is)") // 正確匹配
    }
}
```

`errors.Is` 會遞迴地展開 error chain 來比較。同理，`errors.As` 用於提取特定型別的 error。

## Defer 陷阱

### 陷阱 13：Defer 的參數在宣告時就已求值

```go
package main

import "fmt"

func main() {
    x := 0
    defer fmt.Println("defer x:", x) // x 在這裡就被求值了，是 0

    x = 42
    fmt.Println("x:", x) // 42
}
// 輸出：
// x: 42
// defer x: 0  ← 不是 42！
```

**如果要延遲求值，用 closure：**

```go
x := 0
defer func() {
    fmt.Println("defer x:", x) // closure 捕獲變數，真正執行時才讀取
}()

x = 42
// 輸出：defer x: 42
```

### 陷阱 14：迴圈中的 Defer 造成資源堆積

```go
// 錯誤：defer 要到函式結束才執行，迴圈中開的檔案全部堆在那裡
func processFiles(filenames []string) error {
    for _, name := range filenames {
        f, err := os.Open(name)
        if err != nil {
            return err
        }
        defer f.Close() // 所有檔案要到函式結束才關閉！
        // ... 處理檔案 ...
    }
    return nil
}

// 正確：抽成獨立函式，讓 defer 在每次迭代結束就執行
func processFiles(filenames []string) error {
    for _, name := range filenames {
        if err := processOneFile(name); err != nil {
            return err
        }
    }
    return nil
}

func processOneFile(name string) error {
    f, err := os.Open(name)
    if err != nil {
        return err
    }
    defer f.Close()
    // ... 處理檔案 ...
    return nil
}
```

如果迴圈處理幾千個檔案，`defer` 在迴圈中會導致 file descriptor 耗盡。

### 陷阱 15：Defer 中的 Error 被忽略

```go
// 錯誤：defer 中的 Close error 被丟棄
func writeFile(name string, data []byte) error {
    f, err := os.Create(name)
    if err != nil {
        return err
    }
    defer f.Close() // Close 的 error 被忽略了

    _, err = f.Write(data)
    return err
}

// 正確：捕獲 defer 中的 error
func writeFile(name string, data []byte) (err error) {
    f, err := os.Create(name)
    if err != nil {
        return err
    }
    defer func() {
        if cerr := f.Close(); err == nil {
            err = cerr // 只在沒有其他 error 時，回報 Close error
        }
    }()

    _, err = f.Write(data)
    return err
}
```

## 系列回顧

恭喜你完成了 **Go 語言從零到 Web 應用**完整系列！讓我們回顧走過的路：

1. **為什麼要學 Go** — 認識 Go 的設計哲學與適用場景
2. **環境安裝與第一支程式** — 建立開發環境
3. **基礎語法** — 變數、流程控制、函式
4. **資料結構** — Array、Slice、Map、Struct
5. **錯誤處理與並行** — Error、Goroutine、Channel
6. **HTTP 基礎** — 用標準庫建立 Web Server
7. **專案結構** — 組織大型 Go 專案
8. **資料庫整合** — SQLite + Repository Pattern
9. **使用者認證** — JWT + Middleware
10. **測試與部署** — 單元測試 + Docker
11. **面試必問問題** — 核心概念深度解析
12. **常見陷阱與反模式** — 開發實戰避坑指南（本篇）

每個陷阱的共同主題：**Go 追求簡潔和顯式，但簡潔不代表沒有複雜度**。理解這些底層行為，能讓你寫出更可靠的 Go 程式。

## 下一步

基礎篇和面試/陷阱篇到此告一段落，接下來進入進階篇！我們會學習 Web 框架（Gin vs Echo）、ORM 工具（GORM vs sqlx）、PostgreSQL、gRPC、微服務架構，以及 Kubernetes 部署——把你的 Go 技能提升到職場即戰力。
