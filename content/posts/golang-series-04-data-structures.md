+++
title = 'Go 語言系列（四）：資料結構'
date = 2026-02-24T04:00:00+08:00
draft = false
tags = ['Golang', 'Go', '資料結構', 'Struct', 'Interface', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '深入學習 Go 的核心資料結構：Array、Slice、Map、Struct、Interface 和 Method，理解組合優於繼承的設計'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第四篇。上一篇學了基礎語法，這篇要來認識 Go 的核心資料結構。

## Array（陣列）

Array 是固定長度的同型別元素集合。

```go
// 宣告陣列（長度是型別的一部分）
var scores [5]int  // [0, 0, 0, 0, 0]

// 初始化
scores = [5]int{90, 85, 78, 92, 88}

// 簡短宣告
names := [3]string{"Alice", "Bob", "Charlie"}

// 讓編譯器計算長度
primes := [...]int{2, 3, 5, 7, 11}

// 存取元素
fmt.Println(scores[0])  // 90
scores[1] = 95

// 取得長度
fmt.Println(len(scores))  // 5
```

Array 在 Go 中較少直接使用，因為長度固定且是值語意（傳遞時會複製整個陣列）。大多數情況我們用 Slice。

## Slice（切片）

Slice 是 Go 中最常用的集合型別，可以視為動態長度的陣列。

### 建立 Slice

```go
// 從 array 建立
arr := [5]int{1, 2, 3, 4, 5}
s := arr[1:4]  // [2, 3, 4]（左包含，右不包含）

// 直接宣告
nums := []int{10, 20, 30}

// 使用 make（指定長度和容量）
s1 := make([]int, 5)     // 長度 5，容量 5
s2 := make([]int, 3, 10) // 長度 3，容量 10
```

### 長度 vs 容量

```go
s := make([]int, 3, 5)
fmt.Println(len(s))  // 3（目前元素數量）
fmt.Println(cap(s))  // 5（底層陣列的大小）
```

### append：新增元素

```go
nums := []int{1, 2, 3}
nums = append(nums, 4)          // [1, 2, 3, 4]
nums = append(nums, 5, 6, 7)    // [1, 2, 3, 4, 5, 6, 7]

// 合併兩個 slice
more := []int{8, 9}
nums = append(nums, more...)     // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

當 append 超過容量時，Go 會自動分配更大的底層陣列。

### Slice 操作

```go
s := []int{0, 1, 2, 3, 4, 5}

// 切片語法
s[2:4]  // [2, 3]
s[:3]   // [0, 1, 2]
s[3:]   // [3, 4, 5]
s[:]    // [0, 1, 2, 3, 4, 5]（複製參考）

// 刪除元素（刪除 index 2）
s = append(s[:2], s[3:]...)  // [0, 1, 3, 4, 5]

// 複製 slice
src := []int{1, 2, 3}
dst := make([]int, len(src))
copy(dst, src)
```

### 注意：Slice 是參考型別

```go
original := []int{1, 2, 3}
alias := original      // alias 和 original 指向同一底層陣列

alias[0] = 99
fmt.Println(original)  // [99, 2, 3]  ← 被改了！

// 要獨立複本，用 copy
independent := make([]int, len(original))
copy(independent, original)
```

## Map（映射）

Map 是鍵值對的集合，類似其他語言的 Dictionary 或 HashMap。

### 建立 Map

```go
// 使用 make
ages := make(map[string]int)
ages["Alice"] = 30
ages["Bob"] = 25

// 直接初始化
scores := map[string]int{
    "Alice": 95,
    "Bob":   87,
    "Carol": 92,
}
```

### 操作 Map

```go
m := map[string]int{
    "apple":  5,
    "banana": 3,
}

// 讀取
count := m["apple"]  // 5

// 檢查鍵是否存在（重要！）
value, exists := m["orange"]
if exists {
    fmt.Println("Found:", value)
} else {
    fmt.Println("Not found")
}

// 新增或修改
m["cherry"] = 8

// 刪除
delete(m, "banana")

// 取得長度
fmt.Println(len(m))

// 遍歷（順序不保證）
for key, value := range m {
    fmt.Printf("%s: %d\n", key, value)
}
```

### Map 的零值是 nil

```go
var m map[string]int  // m 是 nil
// m["key"] = 1       // panic! 不能寫入 nil map

// 必須先初始化
m = make(map[string]int)
m["key"] = 1  // OK
```

## Struct（結構體）

Struct 是 Go 定義自訂型別的主要方式，類似其他語言的 class（但沒有繼承）。

### 定義和使用

```go
type User struct {
    Name  string
    Email string
    Age   int
}

// 建立實例
u1 := User{
    Name:  "Jack",
    Email: "jack@example.com",
    Age:   30,
}

// 也可以省略欄位名（按順序）
u2 := User{"Alice", "alice@example.com", 25}

// 存取欄位
fmt.Println(u1.Name)  // Jack
u1.Age = 31

// 零值 struct
var u3 User  // Name: "", Email: "", Age: 0
```

### 巢狀 Struct

```go
type Address struct {
    City    string
    Country string
}

type Person struct {
    Name    string
    Age     int
    Address Address  // 巢狀 struct
}

p := Person{
    Name: "Jack",
    Age:  30,
    Address: Address{
        City:    "Taipei",
        Country: "Taiwan",
    },
}

fmt.Println(p.Address.City)  // Taipei
```

### 匿名欄位（嵌入）

```go
type Person struct {
    Name string
    Age  int
    Address  // 匿名嵌入，直接用型別名稱
}

p := Person{
    Name: "Jack",
    Age:  30,
    Address: Address{
        City:    "Taipei",
        Country: "Taiwan",
    },
}

// 可以直接存取嵌入 struct 的欄位
fmt.Println(p.City)  // Taipei（不需要 p.Address.City）
```

### Struct 指標

```go
u := &User{Name: "Jack", Email: "jack@example.com", Age: 30}

// Go 會自動解參考，不需要寫 (*u).Name
fmt.Println(u.Name)  // Jack
```

### Struct 標籤（Tags）

```go
type User struct {
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age,omitempty"`
}
```

標籤是附加的中繼資料，最常見的用途是 JSON 序列化。`omitempty` 表示零值時省略該欄位。我們在做 Web API 時會大量使用。

## Method（方法）

Method 是附加到型別上的函式。

### 值接收者

```go
type Rectangle struct {
    Width  float64
    Height float64
}

// Area 是 Rectangle 的方法
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

// 使用
rect := Rectangle{Width: 10, Height: 5}
fmt.Println(rect.Area())       // 50
fmt.Println(rect.Perimeter())  // 30
```

`(r Rectangle)` 稱為接收者（receiver），等同於其他語言的 `this` 或 `self`。

### 指標接收者

如果方法需要修改接收者的狀態，使用指標接收者：

```go
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

rect := Rectangle{Width: 10, Height: 5}
rect.Scale(2)
fmt.Println(rect.Width)   // 20
fmt.Println(rect.Height)  // 10
```

**什麼時候用指標接收者？**

- 需要修改接收者的狀態
- 接收者是大型 struct（避免複製的開銷）
- 如果型別的任何一個方法使用指標接收者，那所有方法都應該使用指標接收者（一致性）

### 方法的語法糖

```go
rect := Rectangle{Width: 10, Height: 5}

// 以下兩種寫法等價
rect.Scale(2)

// 實際上 Go 自動做了 (&rect).Scale(2)
```

Go 會根據方法的接收者型別自動取地址或解參考。

## Interface（介面）

Interface 定義了一組方法簽名。任何型別只要實作了這些方法，就自動滿足該 interface——不需要明確宣告。

### 定義 Interface

```go
type Shape interface {
    Area() float64
    Perimeter() float64
}
```

### 隱式實作

```go
type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

// Circle 自動滿足 Shape interface
// 不需要寫 "implements Shape" 之類的宣告
```

### 使用 Interface

```go
func printShapeInfo(s Shape) {
    fmt.Printf("Area: %.2f\n", s.Area())
    fmt.Printf("Perimeter: %.2f\n", s.Perimeter())
}

rect := Rectangle{Width: 10, Height: 5}
circle := Circle{Radius: 7}

printShapeInfo(rect)    // 都可以傳入
printShapeInfo(circle)  // 因為都滿足 Shape interface
```

### 空 Interface

`interface{}` 或 Go 1.18+ 的 `any` 可以接受任何型別的值：

```go
func printAnything(v any) {
    fmt.Printf("Type: %T, Value: %v\n", v, v)
}

printAnything(42)
printAnything("hello")
printAnything(true)
```

### 型別斷言

```go
var i any = "hello"

// 型別斷言
s := i.(string)
fmt.Println(s)  // hello

// 安全的型別斷言
s, ok := i.(string)
if ok {
    fmt.Println("It's a string:", s)
}

// type switch
switch v := i.(type) {
case string:
    fmt.Println("String:", v)
case int:
    fmt.Println("Int:", v)
default:
    fmt.Println("Unknown type")
}
```

### 常用的標準庫 Interface

```go
// fmt.Stringer — 自訂印出格式
type Stringer interface {
    String() string
}

type User struct {
    Name string
    Age  int
}

func (u User) String() string {
    return fmt.Sprintf("%s (%d)", u.Name, u.Age)
}

u := User{Name: "Jack", Age: 30}
fmt.Println(u)  // Jack (30)

// error — Go 的錯誤介面
type error interface {
    Error() string
}
```

## 組合（Composition）vs 繼承

Go 沒有繼承，取而代之的是組合。

### 其他語言的繼承

```python
# Python 的繼承
class Animal:
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"
```

### Go 的組合

```go
type Animal struct {
    Name string
}

func (a Animal) Describe() string {
    return fmt.Sprintf("I am %s", a.Name)
}

type Dog struct {
    Animal     // 嵌入 Animal
    Breed string
}

func (d Dog) Speak() string {
    return "Woof!"
}

dog := Dog{
    Animal: Animal{Name: "Buddy"},
    Breed:  "Labrador",
}

fmt.Println(dog.Describe())  // I am Buddy（提升的方法）
fmt.Println(dog.Speak())     // Woof!
fmt.Println(dog.Name)        // Buddy（提升的欄位）
```

組合的優勢：

- **更靈活**：可以嵌入多個型別
- **更明確**：沒有複雜的繼承層級
- **更容易理解**：看型別定義就知道它有什麼能力

## 練習：建立一個簡單的待辦清單

```go
package main

import "fmt"

type Todo struct {
    ID    int
    Title string
    Done  bool
}

func (t Todo) String() string {
    status := "[ ]"
    if t.Done {
        status = "[x]"
    }
    return fmt.Sprintf("%s %d. %s", status, t.ID, t.Title)
}

type TodoList struct {
    todos  []Todo
    nextID int
}

func NewTodoList() *TodoList {
    return &TodoList{nextID: 1}
}

func (tl *TodoList) Add(title string) {
    tl.todos = append(tl.todos, Todo{
        ID:    tl.nextID,
        Title: title,
        Done:  false,
    })
    tl.nextID++
}

func (tl *TodoList) Complete(id int) bool {
    for i := range tl.todos {
        if tl.todos[i].ID == id {
            tl.todos[i].Done = true
            return true
        }
    }
    return false
}

func (tl *TodoList) List() {
    for _, t := range tl.todos {
        fmt.Println(t)
    }
}

func main() {
    list := NewTodoList()
    list.Add("學習 Go 基礎語法")
    list.Add("學習 Go 資料結構")
    list.Add("建立第一個 Web Server")

    list.Complete(1)

    fmt.Println("=== 待辦清單 ===")
    list.List()
}
```

輸出：
```
=== 待辦清單 ===
[x] 1. 學習 Go 基礎語法
[ ] 2. 學習 Go 資料結構
[ ] 3. 建立第一個 Web Server
```

## 下一步

學完了 Go 的核心資料結構，下一篇我們將進入 Go 最有特色的兩個主題：**錯誤處理**和**併發**。這兩個主題是 Go 區別於其他語言的核心設計，也是寫好 Go 程式的關鍵。
