+++
title = 'Go 語言系列（三）：基礎語法'
date = 2026-02-28T10:00:00+08:00
draft = false
tags = ['Golang', 'Go', '語法', '基礎教學', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '學習 Go 的基礎語法：變數宣告、基本型別、條件判斷、迴圈、函式定義與多值回傳、指標概念'
toc = true
weight = 8
+++

> 這是 **Go 語言從零到 Web 應用**系列的第三篇。上一篇我們設定好了開發環境，現在要來學 Go 的基礎語法。

## 變數宣告

Go 有幾種宣告變數的方式。

### var 宣告

```go
// 宣告並指定型別
var name string = "Jack"

// 宣告並讓編譯器推斷型別
var age = 30

// 先宣告，之後再賦值（會有零值）
var score int    // score = 0
var active bool  // active = false
var label string // label = ""
```

### 短變數宣告 :=

```go
// 只能在函式內使用
name := "Jack"
age := 30
pi := 3.14
isReady := true
```

`:=` 是最常用的宣告方式，簡潔直覺。但注意它只能在函式內使用，套件層級的變數必須用 `var`。

### 多變數宣告

```go
// 同時宣告多個變數
var x, y, z int = 1, 2, 3

// 短變數宣告也可以
a, b, c := "hello", 42, true

// 分組宣告
var (
    firstName string = "Jack"
    lastName  string = "Tse"
    age       int    = 30
)
```

### 常數

```go
const Pi = 3.14159
const MaxRetries = 3

// 分組宣告
const (
    StatusOK    = 200
    StatusNotFound = 404
    StatusError = 500
)
```

常數在編譯時就確定值，不能被修改。

### iota：自動遞增常數

```go
const (
    Sunday = iota    // 0
    Monday           // 1
    Tuesday          // 2
    Wednesday        // 3
    Thursday         // 4
    Friday           // 5
    Saturday         // 6
)
```

`iota` 在每個 `const` 區塊中從 0 開始，每行自動加 1。非常適合定義列舉值。

## 基本型別

### 數值型別

```go
// 整數
var i int     = 42       // 平台相依（32 或 64 bit）
var i8 int8   = 127      // -128 ~ 127
var i16 int16 = 32767
var i32 int32 = 2147483647
var i64 int64 = 9223372036854775807

// 無符號整數
var u uint     = 42
var u8 uint8   = 255     // 0 ~ 255，等同 byte
var u16 uint16 = 65535
var u32 uint32 = 4294967295
var u64 uint64 = 18446744073709551615

// 浮點數
var f32 float32 = 3.14
var f64 float64 = 3.141592653589793  // 預設用 float64
```

**常用原則**：整數用 `int`，浮點數用 `float64`，除非有特殊需求。

### 字串

```go
// 字串是不可變的 byte 序列
s := "Hello, 世界"

// 取得長度（byte 數量）
fmt.Println(len(s))        // 13（中文字佔 3 bytes）

// 取得字元數量
fmt.Println(utf8.RuneCountInString(s))  // 9

// 字串串接
greeting := "Hello" + ", " + "World"

// 多行字串（反引號）
json := `{
    "name": "Jack",
    "age": 30
}`

// 常用字串操作
strings.Contains(s, "Hello")    // true
strings.HasPrefix(s, "He")      // true
strings.ToUpper("hello")        // "HELLO"
strings.Split("a,b,c", ",")    // ["a", "b", "c"]
strings.TrimSpace("  hi  ")    // "hi"
```

### 布林值

```go
isReady := true
isDone := false

// 布林運算
result := isReady && isDone  // AND
result = isReady || isDone   // OR
result = !isReady            // NOT
```

### 零值（Zero Values）

Go 的每個型別都有零值，宣告但未初始化的變數會自動獲得零值：

| 型別 | 零值 |
|------|------|
| int, float | `0` |
| string | `""` |
| bool | `false` |
| pointer, slice, map, channel, function, interface | `nil` |

## 型別轉換

Go 沒有隱式型別轉換，所有轉換必須明確表達：

```go
i := 42
f := float64(i)    // int → float64
u := uint(i)       // int → uint

f2 := 3.14
i2 := int(f2)      // float64 → int（截斷小數，得到 3）

// 字串與數字的轉換
import "strconv"

s := strconv.Itoa(42)          // int → string: "42"
n, err := strconv.Atoi("42")   // string → int: 42
```

## 條件判斷

### if / else

```go
score := 85

if score >= 90 {
    fmt.Println("A")
} else if score >= 80 {
    fmt.Println("B")
} else if score >= 70 {
    fmt.Println("C")
} else {
    fmt.Println("F")
}
```

Go 的 if 有一個特別的功能——可以在條件前加一個初始化語句：

```go
// err 的作用域僅限於 if/else 區塊內
if err := doSomething(); err != nil {
    fmt.Println("Error:", err)
    return
}
```

這個模式在 Go 中非常常見，特別是在錯誤處理時。

### switch

```go
day := "Monday"

switch day {
case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday":
    fmt.Println("工作日")
case "Saturday", "Sunday":
    fmt.Println("週末")
default:
    fmt.Println("無效的日期")
}
```

Go 的 switch 與其他語言不同：

- **不需要 break**：每個 case 執行完自動跳出
- **可以匹配多個值**：用逗號分隔
- **可以沒有條件**：變成 if/else 的替代

```go
// 無條件 switch（更清晰的 if/else chain）
score := 85

switch {
case score >= 90:
    fmt.Println("A")
case score >= 80:
    fmt.Println("B")
case score >= 70:
    fmt.Println("C")
default:
    fmt.Println("F")
}
```

## 迴圈

Go 只有 `for` 一種迴圈，但它能表達所有迴圈形式。

### 標準 for

```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

### while 形式

```go
count := 0
for count < 10 {
    count++
}
```

### 無限迴圈

```go
for {
    fmt.Println("永遠執行")
    // 用 break 跳出
    break
}
```

### range 遍歷

```go
// 遍歷 slice
nums := []int{10, 20, 30}
for index, value := range nums {
    fmt.Printf("index: %d, value: %d\n", index, value)
}

// 只要 index
for i := range nums {
    fmt.Println(i)
}

// 只要 value（用 _ 忽略 index）
for _, v := range nums {
    fmt.Println(v)
}

// 遍歷字串（得到 rune）
for i, ch := range "Hello, 世界" {
    fmt.Printf("index: %d, char: %c\n", i, ch)
}

// 遍歷 map
m := map[string]int{"a": 1, "b": 2}
for key, value := range m {
    fmt.Printf("%s: %d\n", key, value)
}
```

### break 與 continue

```go
for i := 0; i < 10; i++ {
    if i == 3 {
        continue  // 跳過這次迭代
    }
    if i == 7 {
        break     // 跳出迴圈
    }
    fmt.Println(i)
}
// 輸出：0 1 2 4 5 6
```

## 函式

### 基本函式

```go
func greet(name string) string {
    return "Hello, " + name + "!"
}

// 呼叫
message := greet("Jack")
```

### 多值回傳

Go 的函式可以回傳多個值，這是它最有特色的功能之一：

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}

// 呼叫
result, err := divide(10, 3)
if err != nil {
    fmt.Println("Error:", err)
    return
}
fmt.Println(result)
```

多值回傳最常見的用法就是回傳 `(結果, error)`，這是 Go 錯誤處理的核心模式。

### 命名回傳值

```go
func rectangleArea(width, height float64) (area float64) {
    area = width * height
    return  // 自動回傳命名的變數
}
```

命名回傳值會在函式開始時自動初始化為零值。適合用在回傳值含義需要明確表達的場景。

### 可變參數

```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

// 呼叫
fmt.Println(sum(1, 2, 3))       // 6
fmt.Println(sum(1, 2, 3, 4, 5)) // 15

// 展開 slice
nums := []int{1, 2, 3}
fmt.Println(sum(nums...))        // 6
```

### 函式作為值

Go 中的函式是一等公民（first-class citizen），可以當作值傳遞：

```go
// 將函式賦值給變數
add := func(a, b int) int {
    return a + b
}
fmt.Println(add(3, 4))  // 7

// 函式作為參數
func apply(nums []int, fn func(int) int) []int {
    result := make([]int, len(nums))
    for i, v := range nums {
        result[i] = fn(v)
    }
    return result
}

doubled := apply([]int{1, 2, 3}, func(n int) int {
    return n * 2
})
// doubled = [2, 4, 6]
```

## 指標

指標存放的是記憶體位址。Go 有指標但沒有指標運算（比 C 更安全）。

### 基本用法

```go
x := 42
p := &x     // p 是指向 x 的指標，型別是 *int

fmt.Println(p)   // 印出記憶體位址，如 0xc0000b4008
fmt.Println(*p)  // 解參考，印出 42

*p = 100         // 透過指標修改值
fmt.Println(x)   // 100
```

### 為什麼需要指標？

```go
// 不用指標：函式內的修改不影響外部
func doubleValue(n int) {
    n = n * 2  // 只修改了複本
}

x := 10
doubleValue(x)
fmt.Println(x)  // 還是 10

// 使用指標：函式可以修改外部的值
func doublePointer(n *int) {
    *n = *n * 2
}

y := 10
doublePointer(&y)
fmt.Println(y)  // 20
```

Go 是值傳遞的語言——函式參數是原始值的複本。如果你想讓函式修改外部的值，或者避免複製大型結構體，就需要用指標。

### new 函式

```go
// new 分配記憶體並回傳指標
p := new(int)    // *int，指向零值 0
*p = 42
```

## 格式化輸出

`fmt` 套件提供豐富的格式化選項：

```go
name := "Jack"
age := 30
pi := 3.14159

// 常用格式化動詞
fmt.Printf("%s is %d years old\n", name, age)  // 字串和整數
fmt.Printf("Pi = %.2f\n", pi)                   // 浮點數（2 位小數）
fmt.Printf("Type: %T\n", pi)                    // 印出型別
fmt.Printf("Value: %v\n", pi)                   // 通用格式
fmt.Printf("Hex: %x\n", 255)                    // 十六進位
fmt.Printf("Binary: %b\n", 10)                  // 二進位
fmt.Printf("Quoted: %q\n", "hello")             // 帶引號的字串

// Sprintf 回傳格式化後的字串（不印出）
msg := fmt.Sprintf("%s: %d", name, age)
```

## 練習：溫度轉換器

把學到的語法綜合起來，寫一個簡單的溫度轉換器：

```go
package main

import "fmt"

// 攝氏轉華氏
func celsiusToFahrenheit(c float64) float64 {
    return c*9/5 + 32
}

// 華氏轉攝氏
func fahrenheitToCelsius(f float64) float64 {
    return (f - 32) * 5 / 9
}

func main() {
    temperatures := []float64{0, 25, 37, 100}

    fmt.Println("攝氏 → 華氏")
    fmt.Println("============")
    for _, c := range temperatures {
        f := celsiusToFahrenheit(c)
        fmt.Printf("%.1f°C = %.1f°F\n", c, f)
    }

    fmt.Println()
    fmt.Println("華氏 → 攝氏")
    fmt.Println("============")
    fahrenheits := []float64{32, 77, 98.6, 212}
    for _, f := range fahrenheits {
        c := fahrenheitToCelsius(f)
        fmt.Printf("%.1f°F = %.1f°C\n", f, c)
    }
}
```

執行結果：
```
攝氏 → 華氏
============
0.0°C = 32.0°F
25.0°C = 77.0°F
37.0°C = 98.6°F
100.0°C = 212.0°F

華氏 → 攝氏
============
32.0°F = 0.0°C
77.0°F = 25.0°C
98.6°F = 37.0°C
212.0°F = 100.0°C
```

## 下一步

掌握了基礎語法後，下一篇我們要來學習 Go 的資料結構：Array、Slice、Map、Struct 和 Interface。這些是建構任何 Go 應用的核心組件。
