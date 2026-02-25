+++
title = 'Go 語言系列（十三）：Web 框架 — Gin vs Echo'
date = 2026-02-25T13:00:00+08:00
draft = false
tags = ['Golang', 'Go', 'Gin', 'Echo', 'Web框架', 'REST API', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '對比 Go 兩大 Web 框架 Gin 和 Echo，從路由、中間件、JSON 綁定到錯誤處理，用相同的 API 範例展示兩者差異，幫助你做出選擇'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第十三篇，進入進階篇章。前面我們用標準庫 `net/http` 建構了完整的 Web 應用，現在來看看 Web 框架在標準庫之上提供了什麼價值。

## 為什麼需要 Web 框架？

標準庫 `net/http` 非常強大，但在建構大型應用時會遇到一些不便：

```go
// 標準庫：手動解析路由參數
mux := http.NewServeMux()
mux.HandleFunc("/users/", func(w http.ResponseWriter, r *http.Request) {
    // 需要手動從 URL 提取 ID
    id := strings.TrimPrefix(r.URL.Path, "/users/")
    if id == "" {
        http.Error(w, "missing id", http.StatusBadRequest)
        return
    }
    // 還需要手動判斷 HTTP method
    switch r.Method {
    case http.MethodGet:
        // 處理 GET
    case http.MethodPut:
        // 處理 PUT
    default:
        http.Error(w, "method not allowed", http.StatusMethodNotAllowed)
    }
})
```

Web 框架幫你解決的問題：
- **路由參數**：`/users/:id` 自動解析
- **HTTP method 路由**：`GET /users/:id` 和 `PUT /users/:id` 分開定義
- **請求綁定**：JSON、Query、Form 自動綁定到 struct
- **參數驗證**：struct tag 自動驗證
- **中間件**：標準化的中間件鏈
- **錯誤處理**：集中式錯誤處理

Go 生態中最熱門的兩個框架是 **Gin** 和 **Echo**，讓我們用同一個 API 來對比。

## Gin

安裝：

```bash
go get -u github.com/gin-gonic/gin
```

### 路由與參數

```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default() // 內建 Logger + Recovery 中間件

    // 路由參數
    r.GET("/users/:id", getUser)

    // 路由群組
    api := r.Group("/api/v1")
    {
        api.GET("/books", listBooks)
        api.POST("/books", createBook)
        api.GET("/books/:id", getBook)
        api.PUT("/books/:id", updateBook)
        api.DELETE("/books/:id", deleteBook)
    }

    r.Run(":8080")
}
```

### JSON 綁定與驗證

```go
type CreateBookRequest struct {
    Title  string `json:"title" binding:"required,min=1,max=200"`
    Author string `json:"author" binding:"required"`
    Year   int    `json:"year" binding:"required,gte=1000,lte=2100"`
}

func createBook(c *gin.Context) {
    var req CreateBookRequest

    // ShouldBindJSON 自動解析 JSON 並驗證
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{
            "error": err.Error(),
        })
        return
    }

    // 建立書籍...
    c.JSON(http.StatusCreated, gin.H{
        "id":     1,
        "title":  req.Title,
        "author": req.Author,
        "year":   req.Year,
    })
}
```

### 中間件

```go
// 自定義中間件
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "missing authorization header",
            })
            return
        }

        // 驗證 token...
        userID := "user-123" // 假設從 token 解析出來
        c.Set("userID", userID)
        c.Next() // 繼續執行下一個 handler
    }
}

// 使用中間件
func main() {
    r := gin.Default()

    // 全域中間件
    r.Use(AuthMiddleware())

    // 群組中間件
    admin := r.Group("/admin")
    admin.Use(AdminOnly())
    {
        admin.GET("/stats", getStats)
    }
}
```

### 完整 REST API 範例

```go
package main

import (
    "net/http"
    "strconv"
    "sync"

    "github.com/gin-gonic/gin"
)

type Book struct {
    ID     int    `json:"id"`
    Title  string `json:"title"`
    Author string `json:"author"`
}

var (
    books  = make(map[int]Book)
    nextID = 1
    mu     sync.RWMutex
)

func main() {
    r := gin.Default()

    r.GET("/books", func(c *gin.Context) {
        mu.RLock()
        defer mu.RUnlock()
        result := make([]Book, 0, len(books))
        for _, b := range books {
            result = append(result, b)
        }
        c.JSON(http.StatusOK, result)
    })

    r.POST("/books", func(c *gin.Context) {
        var b Book
        if err := c.ShouldBindJSON(&b); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        mu.Lock()
        b.ID = nextID
        nextID++
        books[b.ID] = b
        mu.Unlock()
        c.JSON(http.StatusCreated, b)
    })

    r.GET("/books/:id", func(c *gin.Context) {
        id, _ := strconv.Atoi(c.Param("id"))
        mu.RLock()
        b, ok := books[id]
        mu.RUnlock()
        if !ok {
            c.JSON(http.StatusNotFound, gin.H{"error": "not found"})
            return
        }
        c.JSON(http.StatusOK, b)
    })

    r.DELETE("/books/:id", func(c *gin.Context) {
        id, _ := strconv.Atoi(c.Param("id"))
        mu.Lock()
        delete(books, id)
        mu.Unlock()
        c.Status(http.StatusNoContent)
    })

    r.Run(":8080")
}
```

## Echo

安裝：

```bash
go get -u github.com/labstack/echo/v4
```

### 路由與參數

```go
package main

import (
    "net/http"
    "github.com/labstack/echo/v4"
    "github.com/labstack/echo/v4/middleware"
)

func main() {
    e := echo.New()

    // 內建中間件
    e.Use(middleware.Logger())
    e.Use(middleware.Recover())

    // 路由參數
    e.GET("/users/:id", getUser)

    // 路由群組
    api := e.Group("/api/v1")
    api.GET("/books", listBooks)
    api.POST("/books", createBook)
    api.GET("/books/:id", getBook)
    api.PUT("/books/:id", updateBook)
    api.DELETE("/books/:id", deleteBook)

    e.Logger.Fatal(e.Start(":8080"))
}
```

### JSON 綁定與驗證

```go
type CreateBookRequest struct {
    Title  string `json:"title" validate:"required,min=1,max=200"`
    Author string `json:"author" validate:"required"`
    Year   int    `json:"year" validate:"required,gte=1000,lte=2100"`
}

func createBook(c echo.Context) error {
    var req CreateBookRequest

    // Bind 自動解析 JSON
    if err := c.Bind(&req); err != nil {
        return echo.NewHTTPError(http.StatusBadRequest, err.Error())
    }

    // Echo 需要額外設定 validator
    if err := c.Validate(&req); err != nil {
        return echo.NewHTTPError(http.StatusBadRequest, err.Error())
    }

    return c.JSON(http.StatusCreated, map[string]interface{}{
        "id":     1,
        "title":  req.Title,
        "author": req.Author,
        "year":   req.Year,
    })
}
```

### 中間件

```go
// 自定義中間件
func AuthMiddleware(next echo.HandlerFunc) echo.HandlerFunc {
    return func(c echo.Context) error {
        token := c.Request().Header.Get("Authorization")
        if token == "" {
            return echo.NewHTTPError(http.StatusUnauthorized, "missing authorization header")
        }

        // 驗證 token...
        userID := "user-123"
        c.Set("userID", userID)
        return next(c) // 繼續執行
    }
}

// 使用中間件
func main() {
    e := echo.New()

    // 全域中間件
    e.Use(AuthMiddleware)

    // 群組中間件
    admin := e.Group("/admin", AdminOnly)
    admin.GET("/stats", getStats)
}
```

### 完整 REST API 範例（同 Gin 功能）

```go
package main

import (
    "net/http"
    "strconv"
    "sync"

    "github.com/labstack/echo/v4"
    "github.com/labstack/echo/v4/middleware"
)

type Book struct {
    ID     int    `json:"id"`
    Title  string `json:"title"`
    Author string `json:"author"`
}

var (
    books  = make(map[int]Book)
    nextID = 1
    mu     sync.RWMutex
)

func main() {
    e := echo.New()
    e.Use(middleware.Logger())
    e.Use(middleware.Recover())

    e.GET("/books", func(c echo.Context) error {
        mu.RLock()
        defer mu.RUnlock()
        result := make([]Book, 0, len(books))
        for _, b := range books {
            result = append(result, b)
        }
        return c.JSON(http.StatusOK, result)
    })

    e.POST("/books", func(c echo.Context) error {
        var b Book
        if err := c.Bind(&b); err != nil {
            return echo.NewHTTPError(http.StatusBadRequest, err.Error())
        }
        mu.Lock()
        b.ID = nextID
        nextID++
        books[b.ID] = b
        mu.Unlock()
        return c.JSON(http.StatusCreated, b)
    })

    e.GET("/books/:id", func(c echo.Context) error {
        id, _ := strconv.Atoi(c.Param("id"))
        mu.RLock()
        b, ok := books[id]
        mu.RUnlock()
        if !ok {
            return echo.NewHTTPError(http.StatusNotFound, "not found")
        }
        return c.JSON(http.StatusOK, b)
    })

    e.DELETE("/books/:id", func(c echo.Context) error {
        id, _ := strconv.Atoi(c.Param("id"))
        mu.Lock()
        delete(books, id)
        mu.Unlock()
        return c.NoContent(http.StatusNoContent)
    })

    e.Logger.Fatal(e.Start(":8080"))
}
```

## Gin vs Echo 對比

| 維度 | Gin | Echo |
|------|-----|------|
| **GitHub Stars** | 80k+ | 30k+ |
| **Handler 簽名** | `func(c *gin.Context)` | `func(c echo.Context) error` |
| **錯誤處理** | 手動寫 response | 回傳 `error`，集中處理 |
| **驗證** | 內建（binding tag） | 需自行整合 validator |
| **效能** | 極快（httprouter 基底） | 極快（自建路由樹） |
| **中間件** | `gin.HandlerFunc` | 標準函式包裝 |
| **路由群組** | `r.Group()` + 大括號 | `e.Group()` |
| **JSON 回應** | `c.JSON(code, obj)` | `return c.JSON(code, obj)` |
| **社群生態** | 較大，第三方套件多 | 中等，核心團隊維護良好 |
| **API 風格** | 較隱式（不回傳 error） | 較顯式（handler 回傳 error） |

### 如何選擇？

**選 Gin 如果：**
- 你的團隊已經熟悉 Gin
- 需要更多第三方生態整合
- 偏好內建的驗證功能

**選 Echo 如果：**
- 你偏好 Go 慣用的 error 回傳模式
- 需要更好的錯誤集中處理
- 喜歡較乾淨的 API 設計

兩者效能差距極小，選擇哪個主要看團隊偏好和 API 風格喜好。

## 下一步

選好了框架，下一步要解決資料庫操作的問題。下一篇我們來比較 Go 的兩大資料庫工具：GORM（全功能 ORM）和 sqlx（SQL-first 輕量擴展）。
