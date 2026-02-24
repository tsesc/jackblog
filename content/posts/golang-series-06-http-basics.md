+++
title = 'Go 語言系列（六）：HTTP 基礎與第一個 Web Server'
date = 2026-03-06T10:00:00+08:00
draft = false
tags = ['Golang', 'Go', 'HTTP', 'Web Server', 'net/http', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '使用 Go 標準庫 net/http 建立第一個 Web Server：Handler、路由、JSON 回應與靜態檔案服務'
toc = true
weight = 5
+++

> 這是 **Go 語言從零到 Web 應用**系列的第六篇，也是實戰篇的開始。從這篇開始，我們要用前五篇學到的 Go 基礎，開始建構一個真正的 Web 應用。

## 為什麼 Go 適合做 Web 開發？

Go 的標準庫 `net/http` 已經是一個生產等級的 HTTP 伺服器。不像其他語言需要額外安裝框架（如 Python 的 Flask、Node.js 的 Express），Go 開箱即用就能處理 HTTP 請求。

這個標準庫的品質高到什麼程度？許多知名的 Go Web 框架（如 Gin、Echo）本質上都是在 `net/http` 之上加了一層便利功能。

## 最簡單的 Web Server

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, World!")
    })

    fmt.Println("Server starting on :8080")
    http.ListenAndServe(":8080", nil)
}
```

執行後用瀏覽器打開 `http://localhost:8080`，你就會看到 "Hello, World!"。

就是這麼簡單——5 行核心程式碼就建好了一個 HTTP 伺服器。

## 理解 Handler

### http.Handler 介面

Go 的 HTTP 處理核心是 `Handler` 介面：

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

任何實作了 `ServeHTTP` 方法的型別都可以處理 HTTP 請求。

### http.HandlerFunc

大多數時候我們不會自己實作 `Handler` 介面，而是用 `HandlerFunc`——它是一個將普通函式轉成 Handler 的型別：

```go
// 定義處理函式
func homeHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Welcome to the homepage!")
}

func aboutHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "About page")
}

func main() {
    http.HandleFunc("/", homeHandler)
    http.HandleFunc("/about", aboutHandler)

    http.ListenAndServe(":8080", nil)
}
```

### 自訂 Handler 結構

```go
type APIHandler struct {
    Version string
}

func (h *APIHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "API Version: %s", h.Version)
}

func main() {
    api := &APIHandler{Version: "1.0"}
    http.Handle("/api", api)
    http.ListenAndServe(":8080", nil)
}
```

## Request 物件

`http.Request` 包含了請求的所有資訊：

```go
func debugHandler(w http.ResponseWriter, r *http.Request) {
    // HTTP 方法
    fmt.Fprintf(w, "Method: %s\n", r.Method)

    // 請求路徑
    fmt.Fprintf(w, "Path: %s\n", r.URL.Path)

    // Query 參數
    fmt.Fprintf(w, "Query: %s\n", r.URL.RawQuery)

    // 取得特定 query 參數
    name := r.URL.Query().Get("name")
    fmt.Fprintf(w, "Name: %s\n", name)

    // 請求標頭
    fmt.Fprintf(w, "User-Agent: %s\n", r.Header.Get("User-Agent"))
    fmt.Fprintf(w, "Content-Type: %s\n", r.Header.Get("Content-Type"))

    // 遠端地址
    fmt.Fprintf(w, "Remote: %s\n", r.RemoteAddr)
}
```

### 讀取 Request Body

```go
func bodyHandler(w http.ResponseWriter, r *http.Request) {
    // 讀取 body
    body, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "Error reading body", http.StatusBadRequest)
        return
    }
    defer r.Body.Close()

    fmt.Fprintf(w, "Body: %s", string(body))
}
```

## Response 物件

`http.ResponseWriter` 用來建構回應：

```go
func responseHandler(w http.ResponseWriter, r *http.Request) {
    // 設定回應標頭
    w.Header().Set("Content-Type", "text/plain; charset=utf-8")
    w.Header().Set("X-Custom-Header", "my-value")

    // 設定狀態碼（必須在寫入 body 之前）
    w.WriteHeader(http.StatusOK)  // 200

    // 寫入回應 body
    w.Write([]byte("Hello!"))
}
```

### 常用的 HTTP 狀態碼

```go
http.StatusOK                  // 200
http.StatusCreated             // 201
http.StatusNoContent           // 204
http.StatusBadRequest          // 400
http.StatusUnauthorized        // 401
http.StatusForbidden           // 403
http.StatusNotFound            // 404
http.StatusMethodNotAllowed    // 405
http.StatusInternalServerError // 500
```

### 快速回應錯誤

```go
func handler(w http.ResponseWriter, r *http.Request) {
    // http.Error 會同時設定 Content-Type、狀態碼和 body
    http.Error(w, "Something went wrong", http.StatusInternalServerError)
}
```

## 路由

### 基本路由

```go
mux := http.NewServeMux()

mux.HandleFunc("/", homeHandler)
mux.HandleFunc("/about", aboutHandler)
mux.HandleFunc("/contact", contactHandler)

http.ListenAndServe(":8080", mux)
```

### Go 1.22+ 增強路由

Go 1.22 大幅增強了 `ServeMux` 的路由功能：

```go
mux := http.NewServeMux()

// HTTP 方法限定
mux.HandleFunc("GET /users", listUsersHandler)
mux.HandleFunc("POST /users", createUserHandler)

// 路徑參數
mux.HandleFunc("GET /users/{id}", getUserHandler)
mux.HandleFunc("PUT /users/{id}", updateUserHandler)
mux.HandleFunc("DELETE /users/{id}", deleteUserHandler)

// 在 handler 中取得路徑參數
func getUserHandler(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    fmt.Fprintf(w, "User ID: %s", id)
}
```

這讓我們不需要第三方路由器就能建構 RESTful API。

### 根據 Method 分發

在 Go 1.22 之前，你需要手動檢查 HTTP 方法：

```go
func usersHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        // 列出使用者
    case http.MethodPost:
        // 建立使用者
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}
```

Go 1.22+ 之後，直接在路由定義中指定方法更加簡潔。

## JSON 回應

Web API 最常使用 JSON 格式。Go 的 `encoding/json` 標準庫讓 JSON 處理非常簡單。

### 回傳 JSON

```go
type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func getUserHandler(w http.ResponseWriter, r *http.Request) {
    user := User{
        ID:    1,
        Name:  "Jack",
        Email: "jack@example.com",
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}
```

### JSON 輔助函式

建立一個通用的 JSON 回應函式，之後會經常用到：

```go
func writeJSON(w http.ResponseWriter, status int, data any) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func writeError(w http.ResponseWriter, status int, message string) {
    writeJSON(w, status, map[string]string{"error": message})
}

// 使用
func handler(w http.ResponseWriter, r *http.Request) {
    user := User{ID: 1, Name: "Jack", Email: "jack@example.com"}
    writeJSON(w, http.StatusOK, user)
}
```

### 解析 JSON 請求

```go
func createUserHandler(w http.ResponseWriter, r *http.Request) {
    var user User

    // 解碼 request body
    err := json.NewDecoder(r.Body).Decode(&user)
    if err != nil {
        writeError(w, http.StatusBadRequest, "Invalid JSON")
        return
    }

    // 簡單驗證
    if user.Name == "" {
        writeError(w, http.StatusBadRequest, "Name is required")
        return
    }

    // 假設建立成功，回傳帶有 ID 的使用者
    user.ID = 1
    writeJSON(w, http.StatusCreated, user)
}
```

## 靜態檔案服務

### 提供靜態檔案

```go
// 提供 ./static 目錄下的檔案
fs := http.FileServer(http.Dir("./static"))
mux.Handle("/static/", http.StripPrefix("/static/", fs))
```

目錄結構：
```
project/
├── main.go
└── static/
    ├── style.css
    └── script.js
```

存取 `http://localhost:8080/static/style.css` 就會回傳 `static/style.css` 的內容。

## 完整範例：簡易 API Server

把以上所有概念整合起來：

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "sync"
)

// --- 資料模型 ---

type Book struct {
    ID     int    `json:"id"`
    Title  string `json:"title"`
    Author string `json:"author"`
}

// --- 簡易記憶體儲存 ---

type BookStore struct {
    mu     sync.Mutex
    books  []Book
    nextID int
}

func NewBookStore() *BookStore {
    return &BookStore{nextID: 1}
}

func (s *BookStore) All() []Book {
    s.mu.Lock()
    defer s.mu.Unlock()
    return append([]Book{}, s.books...)
}

func (s *BookStore) Add(b Book) Book {
    s.mu.Lock()
    defer s.mu.Unlock()
    b.ID = s.nextID
    s.nextID++
    s.books = append(s.books, b)
    return b
}

// --- JSON 輔助函式 ---

func writeJSON(w http.ResponseWriter, status int, data any) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func writeError(w http.ResponseWriter, status int, message string) {
    writeJSON(w, status, map[string]string{"error": message})
}

// --- Handler ---

func main() {
    store := NewBookStore()

    // 加入一些初始資料
    store.Add(Book{Title: "The Go Programming Language", Author: "Donovan & Kernighan"})
    store.Add(Book{Title: "Go in Action", Author: "Kennedy, Ketelsen & Martin"})

    mux := http.NewServeMux()

    // GET /books — 列出所有書籍
    mux.HandleFunc("GET /books", func(w http.ResponseWriter, r *http.Request) {
        books := store.All()
        writeJSON(w, http.StatusOK, books)
    })

    // POST /books — 新增書籍
    mux.HandleFunc("POST /books", func(w http.ResponseWriter, r *http.Request) {
        var book Book
        if err := json.NewDecoder(r.Body).Decode(&book); err != nil {
            writeError(w, http.StatusBadRequest, "Invalid JSON")
            return
        }

        if book.Title == "" || book.Author == "" {
            writeError(w, http.StatusBadRequest, "Title and author are required")
            return
        }

        created := store.Add(book)
        writeJSON(w, http.StatusCreated, created)
    })

    // GET / — 首頁
    mux.HandleFunc("GET /", func(w http.ResponseWriter, r *http.Request) {
        writeJSON(w, http.StatusOK, map[string]string{
            "message": "Welcome to the Book API",
            "version": "1.0",
        })
    })

    addr := ":8080"
    fmt.Printf("Server starting on %s\n", addr)
    log.Fatal(http.ListenAndServe(addr, mux))
}
```

### 測試 API

```bash
# 取得首頁
curl http://localhost:8080/

# 列出所有書籍
curl http://localhost:8080/books

# 新增書籍
curl -X POST http://localhost:8080/books \
  -H "Content-Type: application/json" \
  -d '{"title":"Learning Go","author":"Jon Bodner"}'

# 再次列出（應該有 3 本）
curl http://localhost:8080/books
```

## 伺服器設定最佳實踐

在生產環境中，不建議直接使用 `http.ListenAndServe`，而是建立自訂的 `http.Server`：

```go
server := &http.Server{
    Addr:         ":8080",
    Handler:      mux,
    ReadTimeout:  15 * time.Second,
    WriteTimeout: 15 * time.Second,
    IdleTimeout:  60 * time.Second,
}

log.Fatal(server.ListenAndServe())
```

設定超時可以防止慢速攻擊（Slowloris）等安全問題。

## 下一步

我們已經建立了第一個能運作的 API Server！但程式碼目前都塞在一個檔案裡。下一篇我們將學習如何組織 Go Web 專案的目錄結構、使用中間件、以及更進階的路由技巧。
