+++
title = 'Go 語言系列（十）：測試與部署'
date = 2026-03-14T10:00:00+08:00
draft = false
tags = ['Golang', 'Go', '測試', 'Docker', '部署', 'httptest', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '學習 Go 的測試工具鏈：單元測試、表格驅動測試、HTTP Handler 測試，以及使用 Docker 容器化部署應用'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的最後一篇。前九篇我們從零開始建構了一個具備 CRUD、資料庫、認證的完整 Web 應用。這篇要學如何測試和部署它。

## Go 的測試工具

Go 有一個內建的測試框架，不需要安裝任何第三方套件。這是 Go 「電池已附」哲學的體現。

### 基本規則

- 測試檔案以 `_test.go` 結尾
- 測試函式以 `Test` 開頭
- 接受一個 `*testing.T` 參數
- 用 `go test` 執行

## 單元測試

### 基本範例

假設有一個 `math.go`：

```go
package math

func Add(a, b int) int {
    return a + b
}

func Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("division by zero")
    }
    return a / b, nil
}
```

對應的 `math_test.go`：

```go
package math

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Add(2, 3) = %d, want 5", result)
    }
}

func TestDivide(t *testing.T) {
    result, err := Divide(10, 2)
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if result != 5.0 {
        t.Errorf("Divide(10, 2) = %f, want 5.0", result)
    }
}

func TestDivideByZero(t *testing.T) {
    _, err := Divide(10, 0)
    if err == nil {
        t.Fatal("expected error for division by zero")
    }
}
```

### 常用的 testing.T 方法

```go
t.Error("message")    // 記錄錯誤但繼續執行
t.Errorf("format %d", val)  // 格式化版本
t.Fatal("message")    // 記錄錯誤並立即停止這個測試
t.Fatalf("format %d", val)  // 格式化版本
t.Log("message")      // 記錄訊息（只在 -v 模式顯示）
t.Skip("reason")      // 跳過這個測試
```

### 執行測試

```bash
# 執行當前目錄的測試
go test

# 執行所有套件的測試
go test ./...

# 詳細輸出
go test -v ./...

# 執行特定測試
go test -run TestDivide ./...

# 顯示覆蓋率
go test -cover ./...

# 生成覆蓋率報告
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

## 表格驅動測試

表格驅動測試是 Go 社群最推薦的測試風格。將測試案例定義為表格，用迴圈逐一測試：

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -1, -2, -3},
        {"zero", 0, 0, 0},
        {"mixed", -1, 5, 4},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

`t.Run` 建立子測試，每個子測試有自己的名稱，在輸出中容易辨識：

```
=== RUN   TestAdd
=== RUN   TestAdd/positive_numbers
=== RUN   TestAdd/negative_numbers
=== RUN   TestAdd/zero
=== RUN   TestAdd/mixed
--- PASS: TestAdd (0.00s)
```

### 測試驗證邏輯

```go
func TestCreateBookRequest_Validate(t *testing.T) {
    tests := []struct {
        name       string
        req        model.CreateBookRequest
        wantErrors []string
    }{
        {
            name:       "valid request",
            req:        model.CreateBookRequest{Title: "Go Book", Author: "Jack"},
            wantErrors: nil,
        },
        {
            name:       "missing title",
            req:        model.CreateBookRequest{Author: "Jack"},
            wantErrors: []string{"title"},
        },
        {
            name:       "missing both",
            req:        model.CreateBookRequest{},
            wantErrors: []string{"title", "author"},
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            errs := tt.req.Validate()

            if tt.wantErrors == nil {
                if len(errs) != 0 {
                    t.Errorf("expected no errors, got %v", errs)
                }
                return
            }

            for _, field := range tt.wantErrors {
                if _, ok := errs[field]; !ok {
                    t.Errorf("expected error for field %q", field)
                }
            }
        })
    }
}
```

## HTTP Handler 測試

Go 的 `net/http/httptest` 套件提供了測試 HTTP handler 的工具。

### 基本 Handler 測試

```go
import (
    "net/http"
    "net/http/httptest"
    "testing"
)

func TestHomeHandler(t *testing.T) {
    // 建立 request
    req := httptest.NewRequest(http.MethodGet, "/", nil)

    // 建立 response recorder
    w := httptest.NewRecorder()

    // 呼叫 handler
    homeHandler(w, req)

    // 檢查結果
    res := w.Result()
    defer res.Body.Close()

    if res.StatusCode != http.StatusOK {
        t.Errorf("status = %d, want %d", res.StatusCode, http.StatusOK)
    }
}
```

### 測試 JSON API

```go
func TestListBooks(t *testing.T) {
    // 設定 store 和 handler
    store := store.NewMemoryBookStore()
    store.Create(model.CreateBookRequest{Title: "Test Book", Author: "Author"})

    handler := handler.NewBookHandler(store)

    // 建立 request
    req := httptest.NewRequest(http.MethodGet, "/books", nil)
    w := httptest.NewRecorder()

    // 呼叫 handler
    handler.List(w, req)

    // 檢查狀態碼
    if w.Code != http.StatusOK {
        t.Fatalf("status = %d, want %d", w.Code, http.StatusOK)
    }

    // 解析 JSON 回應
    var books []model.Book
    if err := json.NewDecoder(w.Body).Decode(&books); err != nil {
        t.Fatalf("decoding response: %v", err)
    }

    if len(books) != 1 {
        t.Errorf("got %d books, want 1", len(books))
    }

    if books[0].Title != "Test Book" {
        t.Errorf("title = %q, want %q", books[0].Title, "Test Book")
    }
}
```

### 測試 POST 請求

```go
func TestCreateBook(t *testing.T) {
    store := store.NewMemoryBookStore()
    h := handler.NewBookHandler(store)

    body := `{"title":"New Book","author":"New Author"}`
    req := httptest.NewRequest(http.MethodPost, "/books", strings.NewReader(body))
    req.Header.Set("Content-Type", "application/json")

    w := httptest.NewRecorder()
    h.Create(w, req)

    if w.Code != http.StatusCreated {
        t.Fatalf("status = %d, want %d", w.Code, http.StatusCreated)
    }

    var book model.Book
    json.NewDecoder(w.Body).Decode(&book)

    if book.Title != "New Book" {
        t.Errorf("title = %q, want %q", book.Title, "New Book")
    }
    if book.ID == 0 {
        t.Error("expected book to have an ID")
    }
}
```

### 測試認證中間件

```go
func TestAuthMiddleware(t *testing.T) {
    jwtManager := auth.NewJWTManager("test-secret", time.Hour)

    tests := []struct {
        name       string
        authHeader string
        wantStatus int
    }{
        {
            name:       "no header",
            authHeader: "",
            wantStatus: http.StatusUnauthorized,
        },
        {
            name:       "invalid format",
            authHeader: "InvalidToken",
            wantStatus: http.StatusUnauthorized,
        },
        {
            name:       "invalid token",
            authHeader: "Bearer invalid.token.here",
            wantStatus: http.StatusUnauthorized,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // 被保護的 handler
            protected := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
                w.WriteHeader(http.StatusOK)
            })

            handler := middleware.Auth(jwtManager)(protected)

            req := httptest.NewRequest(http.MethodGet, "/", nil)
            if tt.authHeader != "" {
                req.Header.Set("Authorization", tt.authHeader)
            }

            w := httptest.NewRecorder()
            handler.ServeHTTP(w, req)

            if w.Code != tt.wantStatus {
                t.Errorf("status = %d, want %d", w.Code, tt.wantStatus)
            }
        })
    }

    // 測試有效 token
    t.Run("valid token", func(t *testing.T) {
        token, _ := jwtManager.Generate(1, "test@example.com")

        protected := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            userID, ok := middleware.GetUserID(r.Context())
            if !ok || userID != 1 {
                t.Errorf("userID = %d, ok = %v", userID, ok)
            }
            w.WriteHeader(http.StatusOK)
        })

        handler := middleware.Auth(jwtManager)(protected)
        req := httptest.NewRequest(http.MethodGet, "/", nil)
        req.Header.Set("Authorization", "Bearer "+token)

        w := httptest.NewRecorder()
        handler.ServeHTTP(w, req)

        if w.Code != http.StatusOK {
            t.Errorf("status = %d, want %d", w.Code, http.StatusOK)
        }
    })
}
```

### 使用 httptest.Server 做整合測試

```go
func TestAPIIntegration(t *testing.T) {
    // 設定真實的 handler
    mux := setupRouter()  // 你的路由設定函式

    // 啟動測試伺服器
    ts := httptest.NewServer(mux)
    defer ts.Close()

    // 發送真實的 HTTP 請求
    resp, err := http.Get(ts.URL + "/books")
    if err != nil {
        t.Fatal(err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        t.Errorf("status = %d, want %d", resp.StatusCode, http.StatusOK)
    }
}
```

## TestMain：測試前後的設定

```go
func TestMain(m *testing.M) {
    // 測試前的設定
    db := setupTestDB()

    // 執行所有測試
    code := m.Run()

    // 測試後的清理
    db.Close()
    os.Remove("test.db")

    os.Exit(code)
}
```

## Docker 化部署

### Dockerfile

```dockerfile
# Build stage
FROM golang:1.22-alpine AS builder

WORKDIR /app

# 先複製依賴檔案（利用 Docker cache）
COPY go.mod go.sum ./
RUN go mod download

# 複製原始碼並編譯
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server ./cmd/server

# Production stage
FROM alpine:3.19

RUN apk --no-cache add ca-certificates tzdata

WORKDIR /app

# 從 builder 複製執行檔
COPY --from=builder /app/server .

# 非 root 使用者
RUN adduser -D -g '' appuser
USER appuser

EXPOSE 8080

CMD ["./server"]
```

### 多階段建構的好處

- **Build stage**：包含 Go 工具鏈（~1GB）
- **Production stage**：只有編譯後的二進位檔和必要的系統庫（~20MB）

### .dockerignore

```
.git
.env
*.db
tmp/
vendor/
```

### 建構和執行

```bash
# 建構映像
docker build -t bookapi .

# 執行
docker run -p 8080:8080 \
  -e JWT_SECRET=your-secret-key \
  -e PORT=8080 \
  bookapi

# 掛載資料庫檔案（資料持久化）
docker run -p 8080:8080 \
  -v $(pwd)/data:/app/data \
  -e JWT_SECRET=your-secret-key \
  -e DB_PATH=/app/data/bookapi.db \
  bookapi
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      - PORT=8080
      - JWT_SECRET=${JWT_SECRET}
      - DB_PATH=/app/data/bookapi.db
    volumes:
      - db-data:/app/data
    restart: unless-stopped

volumes:
  db-data:
```

```bash
docker compose up -d
```

## 健康檢查端點

在 main.go 中加入健康檢查端點，方便監控和容器編排：

```go
mux.HandleFunc("GET /health", func(w http.ResponseWriter, r *http.Request) {
    // 檢查資料庫連線
    if err := db.Ping(); err != nil {
        writeJSON(w, http.StatusServiceUnavailable, map[string]string{
            "status": "unhealthy",
            "error":  "database connection failed",
        })
        return
    }

    writeJSON(w, http.StatusOK, map[string]string{
        "status": "healthy",
    })
})
```

在 Dockerfile 中加入健康檢查：

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://localhost:8080/health || exit 1
```

## 系列回顧

恭喜你完成了整個系列！讓我們回顧一下學到了什麼：

### 基礎篇

| 篇 | 主題 | 關鍵知識 |
|----|------|----------|
| 1 | 為什麼學 Go | 設計哲學、適用場景、業界應用 |
| 2 | 環境設定 | 安裝、VS Code、Go Modules |
| 3 | 基礎語法 | 變數、型別、流程控制、函式 |
| 4 | 資料結構 | Slice、Map、Struct、Interface |
| 5 | 錯誤處理與併發 | error、Goroutine、Channel |

### 實戰篇

| 篇 | 主題 | 關鍵知識 |
|----|------|----------|
| 6 | HTTP 基礎 | net/http、Handler、JSON |
| 7 | 專案結構 | 目錄結構、中間件、路由 |
| 8 | 資料庫 | database/sql、SQLite、CRUD |
| 9 | 認證 | bcrypt、JWT、保護端點 |
| 10 | 測試與部署 | testing、httptest、Docker |

### 你建構了什麼

一個完整的 **Book API** Web 應用：

- RESTful API（CRUD 操作）
- SQLite 資料庫（持久化儲存）
- 使用者註冊和登入（JWT 認證）
- 中間件（日誌、Recovery、認證）
- 單元測試和整合測試
- Docker 容器化部署

### 繼續學習的方向

1. **Web 框架**：試試 Gin 或 Echo，了解它們在標準庫之上提供了什麼
2. **ORM**：學習 GORM 或 sqlx，簡化資料庫操作
3. **PostgreSQL**：將 SQLite 換成 PostgreSQL，學習更進階的資料庫功能
4. **gRPC**：學習 Go 的另一個強項——高效能 RPC 框架
5. **微服務**：學習如何將單體應用拆分成微服務
6. **Kubernetes**：學習如何在 K8s 上部署 Go 應用

Go 的世界很大，但你已經有了穩固的基礎。保持好奇心，持續寫程式，你會越來越熟練。

Happy Coding!
