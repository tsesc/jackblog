+++
title = 'Go 語言系列（十一）：面試必問問題'
date = 2026-02-25T11:00:00+08:00
draft = false
tags = ['Golang', 'Go', '面試', 'Goroutine', 'Channel', 'Interface', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '整理 Go 語言面試中最常被問到的核心概念：Goroutine 排程、記憶體管理、Interface 底層、Slice 原理、Map 實作與 Defer 機制，每題附程式碼範例與深度解析'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第十一篇。前十篇我們完成了從基礎到實戰的完整學習路徑，這篇整理 Go 面試中最常被問到的核心問題，幫助你在面試中展現對 Go 的深入理解。

## Goroutine 與 Channel

### Q1：Go 的 Goroutine 排程模型是什麼？

Go 使用 **GMP 模型**來排程 goroutine：

- **G（Goroutine）**：要執行的工作單元，每個 `go func()` 建立一個 G
- **M（Machine）**：作業系統執行緒，實際執行程式碼的載體
- **P（Processor）**：邏輯處理器，持有本地的 G 佇列，數量預設等於 CPU 核心數

排程流程：

```
G1, G2, G3 ...  →  P（本地佇列）→  M（OS Thread）→  CPU
```

P 從自己的本地佇列取出 G 放到 M 上執行。當本地佇列空了，會從其他 P「偷」G 來執行（work stealing）。

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    // 查看 P 的數量
    fmt.Println("GOMAXPROCS:", runtime.GOMAXPROCS(0))

    // 查看目前的 goroutine 數量
    fmt.Println("NumGoroutine:", runtime.NumGoroutine())

    for i := 0; i < 5; i++ {
        go func(n int) {
            fmt.Printf("goroutine %d running on thread\n", n)
        }(i)
    }

    fmt.Println("NumGoroutine after launch:", runtime.NumGoroutine())
    runtime.Gosched() // 讓出 CPU 給其他 goroutine
}
```

**面試加分點：**
- Goroutine 的 stack 初始只有 2KB（OS thread 通常是 1-8MB），可以動態成長
- `runtime.GOMAXPROCS(n)` 可以設定 P 的數量
- M 的數量可以超過 P，例如當 goroutine 進行系統呼叫被阻塞時，runtime 會建立新的 M

### Q2：Unbuffered 和 Buffered Channel 有什麼差異？

```go
package main

import "fmt"

func main() {
    // Unbuffered channel：發送和接收必須同時就緒
    ch1 := make(chan int)

    go func() {
        ch1 <- 42 // 會阻塞，直到有人接收
    }()
    val := <-ch1 // 接收，此時發送方才會解除阻塞
    fmt.Println("unbuffered:", val)

    // Buffered channel：在 buffer 滿之前，發送不會阻塞
    ch2 := make(chan int, 2)
    ch2 <- 1 // 不阻塞，buffer 還有空間
    ch2 <- 2 // 不阻塞，buffer 剛好滿
    // ch2 <- 3 // 這裡會阻塞！buffer 已滿

    fmt.Println("buffered:", <-ch2, <-ch2)
}
```

**關鍵差異：**

| 特性 | Unbuffered | Buffered |
|------|-----------|----------|
| 建立方式 | `make(chan T)` | `make(chan T, n)` |
| 發送阻塞 | 直到有接收者 | 直到 buffer 滿 |
| 接收阻塞 | 直到有發送者 | 直到 buffer 空 |
| 同步語義 | 同步（rendezvous） | 非同步（有 buffer 空間時） |

### Q3：Select 語句怎麼用？多個 case 同時就緒會怎樣？

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(100 * time.Millisecond)
        ch1 <- "from ch1"
    }()
    go func() {
        time.Sleep(200 * time.Millisecond)
        ch2 <- "from ch2"
    }()

    // select 會等待任一 case 就緒
    select {
    case msg := <-ch1:
        fmt.Println(msg)
    case msg := <-ch2:
        fmt.Println(msg)
    case <-time.After(1 * time.Second):
        fmt.Println("timeout")
    }
}
```

**面試重點：**
- 多個 case 同時就緒時，Go 會**隨機**選擇一個（不是按順序）
- `default` case 讓 select 變成非阻塞
- 常用於實現 timeout、fan-in、done channel 等模式

## 記憶體管理

### Q4：Go 的變數何時分配在 Stack，何時在 Heap？

Go 的編譯器透過 **escape analysis** 決定變數分配在 stack 還是 heap：

```go
package main

import "fmt"

// p 逃逸到 heap，因為函式回傳了指向它的 pointer
func newInt() *int {
    p := 42
    return &p // p escapes to heap
}

// x 留在 stack，因為沒有逃逸
func addOne(x int) int {
    x++
    return x // x stays on stack
}

func main() {
    ptr := newInt()
    fmt.Println(*ptr)

    result := addOne(10)
    fmt.Println(result)
}
```

用 `go build -gcflags="-m"` 檢查 escape analysis：

```bash
$ go build -gcflags="-m" main.go
# command-line-arguments
./main.go:7:2: moved to heap: p
./main.go:18:13: ... argument does not escape
```

**常見逃逸場景：**
- 函式回傳 local 變數的 pointer
- 變數存入 interface（編譯器無法確定大小）
- 變數被 closure 捕獲
- 變數太大（超過 stack 限制）

### Q5：Go 的 Garbage Collector 是如何運作的？

Go 使用 **concurrent, tri-color, mark-and-sweep** GC：

**三色標記法：**
1. **白色**：未被掃描的物件（GC 結束後被回收）
2. **灰色**：已被發現但子物件還沒掃完
3. **黑色**：已完成掃描的物件（存活）

```
初始狀態：所有物件為白色
         ↓
從 root 開始，標記直接可達的物件為灰色
         ↓
取一個灰色物件，掃描它引用的物件（標灰），自己變黑
         ↓
重複直到沒有灰色物件
         ↓
回收所有白色物件
```

**面試加分點：**
- Go 的 GC 是 **concurrent** 的，和應用程式同時運行，減少 STW（Stop The World）時間
- 使用 **write barrier** 確保併發標記的正確性
- `GOGC` 環境變數控制 GC 頻率（預設 100，表示 heap 成長 100% 觸發 GC）
- Go 1.19+ 新增 `GOMEMLIMIT` 可以設定記憶體上限

## Interface

### Q6：Interface 的底層結構是什麼？

Go 的 interface 有兩種內部表示：

```go
// eface：空 interface（interface{}）
type eface struct {
    _type *_type // 型別資訊
    data  unsafe.Pointer // 指向實際資料
}

// iface：帶方法的 interface
type iface struct {
    tab  *itab          // 型別 + 方法表
    data unsafe.Pointer // 指向實際資料
}
```

```go
package main

import "fmt"

type Animal interface {
    Speak() string
}

type Dog struct {
    Name string
}

func (d Dog) Speak() string {
    return d.Name + " says woof!"
}

func main() {
    // iface：帶方法的 interface
    var a Animal = Dog{Name: "Buddy"}
    fmt.Println(a.Speak())

    // eface：空 interface
    var e interface{} = 42
    fmt.Println(e)
}
```

`iface` 中的 `itab` 包含了具體型別的方法表（vtable），這就是 Go 實現多型的方式。每個 (interface type, concrete type) 組合有一個 `itab`，會被 runtime 快取。

### Q7：Type Assertion 和 Type Switch 怎麼用？

```go
package main

import "fmt"

type Shape interface {
    Area() float64
}

type Circle struct{ Radius float64 }
type Rect struct{ Width, Height float64 }

func (c Circle) Area() float64 { return 3.14159 * c.Radius * c.Radius }
func (r Rect) Area() float64   { return r.Width * r.Height }

func describe(s Shape) {
    // Type assertion：斷言具體型別
    if c, ok := s.(Circle); ok {
        fmt.Printf("Circle with radius %.1f\n", c.Radius)
        return
    }

    // Type switch：多型別判斷
    switch v := s.(type) {
    case Circle:
        fmt.Printf("Circle: %.2f\n", v.Area())
    case Rect:
        fmt.Printf("Rect: %.2f\n", v.Area())
    default:
        fmt.Printf("Unknown shape: %.2f\n", v.Area())
    }
}

func main() {
    describe(Circle{Radius: 5})
    describe(Rect{Width: 3, Height: 4})
}
```

**面試重點：**
- 不帶 `ok` 的 type assertion 失敗會 panic：`v := i.(Type)`
- 帶 `ok` 的不會 panic：`v, ok := i.(Type)`
- Type switch 不能用 `fallthrough`

## Slice

### Q8：Slice 的底層結構是什麼？

Slice 是一個三元組的 header：

```go
type slice struct {
    array unsafe.Pointer // 指向底層 array
    len   int           // 目前長度
    cap   int           // 容量（底層 array 的大小）
}
```

```go
package main

import "fmt"

func main() {
    // 建立一個長度 3、容量 5 的 slice
    s := make([]int, 3, 5)
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
    // len=3 cap=5 [0 0 0]

    // append 不超過 cap，不會分配新 array
    s = append(s, 1)
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
    // len=4 cap=5 [0 0 0 1]

    // 截取 slice 共享底層 array
    s2 := s[1:3]
    fmt.Printf("s2: len=%d cap=%d %v\n", len(s2), cap(s2), s2)
    // s2: len=2 cap=4 [0 0]

    s2[0] = 999
    fmt.Println("s after s2 modification:", s)
    // s after s2 modification: [0 999 0 1] — s 也被改了！
}
```

### Q9：Append 的擴容機制是什麼？

```go
package main

import "fmt"

func main() {
    var s []int
    prev := cap(s)

    for i := 0; i < 20; i++ {
        s = append(s, i)
        if cap(s) != prev {
            fmt.Printf("len=%-2d cap changed: %d → %d\n", len(s), prev, cap(s))
            prev = cap(s)
        }
    }
}
```

輸出（Go 1.21+）：
```
len=1  cap changed: 0 → 1
len=2  cap changed: 1 → 2
len=3  cap changed: 2 → 4
len=5  cap changed: 4 → 8
len=9  cap changed: 8 → 16
len=17 cap changed: 16 → 32
```

**擴容規則（Go 1.18+）：**
- 新容量 < 256：翻倍
- 新容量 >= 256：成長 `(newcap + 3*256) / 4`，約 1.25 倍漸進成長
- 最終會對齊到記憶體分配器的 size class

## Map

### Q10：Map 的底層是怎麼實作的？

Go 的 map 使用 **hash table**，底層結構是 `hmap`：

```go
// 簡化版結構
type hmap struct {
    count     int    // 元素數量
    B         uint8  // 桶數量 = 2^B
    buckets   unsafe.Pointer // 桶陣列
    oldbuckets unsafe.Pointer // 擴容時的舊桶
}
```

每個桶（bucket）存放最多 8 個 key-value pair。當 load factor 超過 6.5 時觸發擴容。

### Q11：為什麼 Map 不是 Thread-Safe？

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    m := make(map[int]int)
    var wg sync.WaitGroup

    // 這段程式碼會 panic: concurrent map writes
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            m[n] = n * n // 併發寫入！
        }(i)
    }
    wg.Wait()
    fmt.Println(m)
}
```

Go 的 map 不加鎖是刻意的設計決策——大部分使用場景不需要併發存取，加鎖會降低效能。

**解決方案：**

```go
// 方案一：sync.Mutex
type SafeMap struct {
    mu sync.RWMutex
    m  map[string]int
}

func (s *SafeMap) Get(key string) (int, bool) {
    s.mu.RLock()
    defer s.mu.RUnlock()
    v, ok := s.m[key]
    return v, ok
}

func (s *SafeMap) Set(key string, val int) {
    s.mu.Lock()
    defer s.mu.Unlock()
    s.m[key] = val
}

// 方案二：sync.Map（適合讀多寫少的場景）
var sm sync.Map
sm.Store("key", "value")
val, ok := sm.Load("key")
```

## Defer、Panic、Recover

### Q12：Defer 的執行順序是什麼？

Defer 遵循 **LIFO（後進先出）** 順序：

```go
package main

import "fmt"

func main() {
    fmt.Println("start")

    defer fmt.Println("first defer")
    defer fmt.Println("second defer")
    defer fmt.Println("third defer")

    fmt.Println("end")
}
// 輸出：
// start
// end
// third defer
// second defer
// first defer
```

### Q13：Defer 如何影響具名回傳值？

```go
package main

import "fmt"

func example() (result int) {
    defer func() {
        result++ // defer 可以修改具名回傳值
    }()

    return 0 // 先設定 result = 0，再執行 defer，result 變成 1
}

func main() {
    fmt.Println(example()) // 1
}
```

**執行順序：**
1. `return 0` → 設定 `result = 0`
2. 執行 defer → `result++`
3. 函式真正回傳 → `result = 1`

### Q14：Panic 和 Recover 的正確用法？

```go
package main

import "fmt"

func safeDivide(a, b int) (result int, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("recovered from panic: %v", r)
        }
    }()

    return a / b, nil // b=0 時會 panic
}

func main() {
    result, err := safeDivide(10, 0)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Result:", result)
}
```

**面試重點：**
- `recover()` 只在 `defer` 函式中有效
- `recover()` 只能捕獲當前 goroutine 的 panic
- 正常的 error 應該用 `error` 回傳值處理，`panic/recover` 只用於不可預期的嚴重錯誤

## 下一步

這篇整理了 Go 面試中最高頻的核心問題。下一篇我們來看 Go 開發中最常踩到的陷阱和反模式——知道這些坑在哪裡，能讓你少走很多彎路。
