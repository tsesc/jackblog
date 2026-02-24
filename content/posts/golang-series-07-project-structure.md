+++
title = 'Go 語言系列（七）：專案結構與路由'
date = 2026-02-24T07:00:00+08:00
draft = false
tags = ['Golang', 'Go', '專案結構', 'Middleware', '路由', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '學習 Go Web 專案的標準目錄結構、中間件設計模式、進階路由技巧與環境變數管理'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第七篇。上一篇我們建立了第一個 API Server，但所有程式碼都在一個檔案裡。這篇要來學如何組織一個可維護的專案結構。

## 專案目錄結構

Go 社群雖然沒有強制的專案結構，但有一個被廣泛採用的慣例。以下是我們的 Web 應用將使用的結構：

```
bookapi/
├── cmd/
│   └── server/
│       └── main.go          # 程式進入點
├── internal/
│   ├── handler/
│   │   └── book.go          # HTTP handler
│   ├── middleware/
│   │   ├── logging.go       # 日誌中間件
│   │   └── recovery.go      # Panic recovery 中間件
│   ├── model/
│   │   └── book.go          # 資料模型
│   └── store/
│       └── book.go          # 資料存取層
├── go.mod
├── go.sum
└── .env                     # 環境變數（不進版控）
```

### 目錄說明

- **`cmd/`**：可執行程式的進入點。每個子目錄對應一個執行檔
- **`internal/`**：私有程式碼，Go 編譯器會阻止外部套件引用這個目錄下的程式碼
- **`handler/`**：HTTP 請求處理器
- **`middleware/`**：中間件
- **`model/`**：資料模型定義
- **`store/`**：資料存取邏輯

## 初始化專案

```bash
mkdir bookapi && cd bookapi
go mod init bookapi
```

## 定義資料模型

### internal/model/book.go

```go
package model

import "time"

type Book struct {
    ID        int       `json:"id"`
    Title     string    `json:"title"`
    Author    string    `json:"author"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

type CreateBookRequest struct {
    Title  string `json:"title"`
    Author string `json:"author"`
}

func (r CreateBookRequest) Validate() map[string]string {
    errors := make(map[string]string)
    if r.Title == "" {
        errors["title"] = "title is required"
    }
    if r.Author == "" {
        errors["author"] = "author is required"
    }
    return errors
}

type UpdateBookRequest struct {
    Title  *string `json:"title"`
    Author *string `json:"author"`
}
```

使用獨立的 Request 型別而非直接用 `Book`，可以明確區分「客戶端送來的資料」和「系統內部的資料」。`UpdateBookRequest` 中的指標欄位讓我們能區分「沒有提供」和「設為空字串」。

## 資料存取層

### internal/store/book.go

```go
package store

import (
    "bookapi/internal/model"
    "errors"
    "sync"
    "time"
)

var ErrNotFound = errors.New("not found")

type BookStore struct {
    mu     sync.RWMutex
    books  []model.Book
    nextID int
}

func NewBookStore() *BookStore {
    return &BookStore{nextID: 1}
}

func (s *BookStore) List() []model.Book {
    s.mu.RLock()
    defer s.mu.RUnlock()
    result := make([]model.Book, len(s.books))
    copy(result, s.books)
    return result
}

func (s *BookStore) GetByID(id int) (model.Book, error) {
    s.mu.RLock()
    defer s.mu.RUnlock()
    for _, b := range s.books {
        if b.ID == id {
            return b, nil
        }
    }
    return model.Book{}, ErrNotFound
}

func (s *BookStore) Create(req model.CreateBookRequest) model.Book {
    s.mu.Lock()
    defer s.mu.Unlock()

    now := time.Now()
    book := model.Book{
        ID:        s.nextID,
        Title:     req.Title,
        Author:    req.Author,
        CreatedAt: now,
        UpdatedAt: now,
    }
    s.nextID++
    s.books = append(s.books, book)
    return book
}

func (s *BookStore) Update(id int, req model.UpdateBookRequest) (model.Book, error) {
    s.mu.Lock()
    defer s.mu.Unlock()

    for i := range s.books {
        if s.books[i].ID == id {
            if req.Title != nil {
                s.books[i].Title = *req.Title
            }
            if req.Author != nil {
                s.books[i].Author = *req.Author
            }
            s.books[i].UpdatedAt = time.Now()
            return s.books[i], nil
        }
    }
    return model.Book{}, ErrNotFound
}

func (s *BookStore) Delete(id int) error {
    s.mu.Lock()
    defer s.mu.Unlock()

    for i := range s.books {
        if s.books[i].ID == id {
            s.books = append(s.books[:i], s.books[i+1:]...)
            return nil
        }
    }
    return ErrNotFound
}
```

注意：

- 使用 `sync.RWMutex` 而非 `sync.Mutex`，讀取操作可以並行
- `ErrNotFound` 是一個 sentinel error，讓 handler 層能判斷是「找不到」還是「其他錯誤」
- 目前用記憶體儲存，下一篇會換成 SQLite

## 中間件（Middleware）

中間件是在 Handler 前後執行的邏輯，用來處理跨 handler 的共通功能。

### 中間件的概念

```
Request → [Logging] → [Recovery] → [Handler] → Response
```

在 Go 中，中間件就是一個接受 `http.Handler` 並回傳 `http.Handler` 的函式：

```go
type Middleware func(http.Handler) http.Handler
```

### internal/middleware/logging.go

```go
package middleware

import (
    "log"
    "net/http"
    "time"
)

type wrappedWriter struct {
    http.ResponseWriter
    statusCode int
}

func (w *wrappedWriter) WriteHeader(statusCode int) {
    w.statusCode = statusCode
    w.ResponseWriter.WriteHeader(statusCode)
}

func Logging(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        wrapped := &wrappedWriter{ResponseWriter: w, statusCode: http.StatusOK}

        next.ServeHTTP(wrapped, r)

        log.Printf(
            "%s %s %d %v",
            r.Method,
            r.URL.Path,
            wrapped.statusCode,
            time.Since(start),
        )
    })
}
```

`wrappedWriter` 包裝了原始的 `ResponseWriter`，讓我們能捕獲 handler 設定的狀態碼。

### internal/middleware/recovery.go

```go
package middleware

import (
    "log"
    "net/http"
)

func Recovery(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                log.Printf("panic recovered: %v", err)
                http.Error(w, "Internal Server Error", http.StatusInternalServerError)
            }
        }()

        next.ServeHTTP(wrapped, r)
    })
}
```

Recovery 中間件防止單一請求的 panic 導致整個伺服器崩潰。

### 串接中間件

```go
// 手動串接
handler := middleware.Logging(middleware.Recovery(mux))

// 或建立一個 Chain 輔助函式
func Chain(handler http.Handler, middlewares ...func(http.Handler) http.Handler) http.Handler {
    for i := len(middlewares) - 1; i >= 0; i-- {
        handler = middlewares[i](handler)
    }
    return handler
}

// 使用
handler := Chain(mux, middleware.Logging, middleware.Recovery)
```

## HTTP Handler

### internal/handler/book.go

```go
package handler

import (
    "bookapi/internal/model"
    "bookapi/internal/store"
    "encoding/json"
    "errors"
    "net/http"
    "strconv"
)

type BookHandler struct {
    store *store.BookStore
}

func NewBookHandler(s *store.BookStore) *BookHandler {
    return &BookHandler{store: s}
}

func (h *BookHandler) RegisterRoutes(mux *http.ServeMux) {
    mux.HandleFunc("GET /books", h.List)
    mux.HandleFunc("POST /books", h.Create)
    mux.HandleFunc("GET /books/{id}", h.Get)
    mux.HandleFunc("PUT /books/{id}", h.Update)
    mux.HandleFunc("DELETE /books/{id}", h.Delete)
}

func (h *BookHandler) List(w http.ResponseWriter, r *http.Request) {
    books := h.store.List()
    writeJSON(w, http.StatusOK, books)
}

func (h *BookHandler) Get(w http.ResponseWriter, r *http.Request) {
    id, err := strconv.Atoi(r.PathValue("id"))
    if err != nil {
        writeError(w, http.StatusBadRequest, "Invalid book ID")
        return
    }

    book, err := h.store.GetByID(id)
    if err != nil {
        if errors.Is(err, store.ErrNotFound) {
            writeError(w, http.StatusNotFound, "Book not found")
            return
        }
        writeError(w, http.StatusInternalServerError, "Internal error")
        return
    }

    writeJSON(w, http.StatusOK, book)
}

func (h *BookHandler) Create(w http.ResponseWriter, r *http.Request) {
    var req model.CreateBookRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeError(w, http.StatusBadRequest, "Invalid JSON")
        return
    }

    if errs := req.Validate(); len(errs) > 0 {
        writeJSON(w, http.StatusBadRequest, map[string]any{
            "error":  "Validation failed",
            "fields": errs,
        })
        return
    }

    book := h.store.Create(req)
    writeJSON(w, http.StatusCreated, book)
}

func (h *BookHandler) Update(w http.ResponseWriter, r *http.Request) {
    id, err := strconv.Atoi(r.PathValue("id"))
    if err != nil {
        writeError(w, http.StatusBadRequest, "Invalid book ID")
        return
    }

    var req model.UpdateBookRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeError(w, http.StatusBadRequest, "Invalid JSON")
        return
    }

    book, err := h.store.Update(id, req)
    if err != nil {
        if errors.Is(err, store.ErrNotFound) {
            writeError(w, http.StatusNotFound, "Book not found")
            return
        }
        writeError(w, http.StatusInternalServerError, "Internal error")
        return
    }

    writeJSON(w, http.StatusOK, book)
}

func (h *BookHandler) Delete(w http.ResponseWriter, r *http.Request) {
    id, err := strconv.Atoi(r.PathValue("id"))
    if err != nil {
        writeError(w, http.StatusBadRequest, "Invalid book ID")
        return
    }

    if err := h.store.Delete(id); err != nil {
        if errors.Is(err, store.ErrNotFound) {
            writeError(w, http.StatusNotFound, "Book not found")
            return
        }
        writeError(w, http.StatusInternalServerError, "Internal error")
        return
    }

    w.WriteHeader(http.StatusNoContent)
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
```

### 路由註冊模式

每個 handler 群組負責註冊自己的路由（`RegisterRoutes`），而非在 main 中集中管理。這樣當 handler 增加時，`main.go` 不會越來越肥。

## 環境變數管理

### 使用 os 套件

```go
import "os"

port := os.Getenv("PORT")
if port == "" {
    port = "8080"
}
```

### 建立 Config 結構

```go
type Config struct {
    Port string
    Env  string
}

func LoadConfig() Config {
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    env := os.Getenv("APP_ENV")
    if env == "" {
        env = "development"
    }

    return Config{
        Port: port,
        Env:  env,
    }
}
```

### .env 檔案

開發時可以建立 `.env` 檔案（記得加到 `.gitignore`）：

```env
PORT=8080
APP_ENV=development
```

使用第三方套件 `github.com/joho/godotenv` 來載入：

```bash
go get github.com/joho/godotenv
```

```go
import "github.com/joho/godotenv"

func init() {
    godotenv.Load()  // 載入 .env 檔案
}
```

## 程式進入點

### cmd/server/main.go

```go
package main

import (
    "bookapi/internal/handler"
    "bookapi/internal/middleware"
    "bookapi/internal/model"
    "bookapi/internal/store"
    "fmt"
    "log"
    "net/http"
    "os"
    "time"
)

func main() {
    // 設定
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    // 初始化 store
    bookStore := store.NewBookStore()

    // 加入範例資料
    bookStore.Create(model.CreateBookRequest{
        Title:  "The Go Programming Language",
        Author: "Donovan & Kernighan",
    })

    // 建立路由
    mux := http.NewServeMux()

    // 註冊 handler
    bookHandler := handler.NewBookHandler(bookStore)
    bookHandler.RegisterRoutes(mux)

    // 首頁
    mux.HandleFunc("GET /", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{"message":"Book API","version":"1.0"}`)
    })

    // 套用中間件
    var h http.Handler = mux
    h = middleware.Logging(h)
    h = middleware.Recovery(h)

    // 建立伺服器
    server := &http.Server{
        Addr:         ":" + port,
        Handler:      h,
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }

    log.Printf("Server starting on :%s", port)
    log.Fatal(server.ListenAndServe())
}
```

### 執行

```bash
go run ./cmd/server
```

## 請求參數解析

### Query Parameters

```go
// GET /books?page=2&limit=10&sort=title
func (h *BookHandler) List(w http.ResponseWriter, r *http.Request) {
    query := r.URL.Query()

    page, _ := strconv.Atoi(query.Get("page"))
    if page < 1 {
        page = 1
    }

    limit, _ := strconv.Atoi(query.Get("limit"))
    if limit < 1 || limit > 100 {
        limit = 10
    }

    sort := query.Get("sort")
    // ...
}
```

### Path Parameters（Go 1.22+）

```go
// GET /books/{id}
id := r.PathValue("id")

// GET /users/{userID}/books/{bookID}
userID := r.PathValue("userID")
bookID := r.PathValue("bookID")
```

### Form Data

```go
func formHandler(w http.ResponseWriter, r *http.Request) {
    // 解析表單資料
    r.ParseForm()

    name := r.FormValue("name")
    email := r.FormValue("email")
    // ...
}
```

## 下一步

現在我們有了一個結構清晰的 Web 專案，但資料存在記憶體中，伺服器重啟就消失了。下一篇我們將整合 SQLite 資料庫，讓資料能持久化儲存。
