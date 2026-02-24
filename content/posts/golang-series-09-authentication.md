+++
title = 'Go 語言系列（九）：使用者認證'
date = 2026-03-12T10:00:00+08:00
draft = false
tags = ['Golang', 'Go', 'JWT', 'bcrypt', '認證', '安全性', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '在 Go Web 應用中實現完整的使用者認證系統：密碼雜湊、JWT Token、認證中間件與受保護的 API 端點'
toc = true
weight = 2
+++

> 這是 **Go 語言從零到 Web 應用**系列的第九篇。上一篇我們整合了資料庫，這篇要加入使用者認證，讓 API 知道「誰」在操作。

## 認證的基本概念

### Authentication vs Authorization

- **Authentication（認證）**：驗證「你是誰」——使用者登入
- **Authorization（授權）**：驗證「你能做什麼」——權限檢查

這篇聚焦在 Authentication。

### JWT 認證流程

我們採用 JWT（JSON Web Token）認證方式：

```
1. 使用者註冊 → 密碼用 bcrypt 雜湊後存入資料庫
2. 使用者登入 → 驗證密碼 → 回傳 JWT Token
3. 後續請求 → 在 Header 帶上 Token → 中間件驗證 → 存取受保護資源
```

## 安裝依賴

```bash
go get golang.org/x/crypto/bcrypt
go get github.com/golang-jwt/jwt/v5
```

- **bcrypt**：密碼雜湊演算法
- **golang-jwt**：Go 社群最廣泛使用的 JWT 套件

## 使用者模型

### internal/model/user.go

```go
package model

import "time"

type User struct {
    ID           int       `json:"id"`
    Username     string    `json:"username"`
    Email        string    `json:"email"`
    PasswordHash string    `json:"-"`  // json:"-" 永遠不會出現在 JSON 回應中
    CreatedAt    time.Time `json:"created_at"`
}

type RegisterRequest struct {
    Username string `json:"username"`
    Email    string `json:"email"`
    Password string `json:"password"`
}

func (r RegisterRequest) Validate() map[string]string {
    errs := make(map[string]string)
    if r.Username == "" {
        errs["username"] = "username is required"
    }
    if r.Email == "" {
        errs["email"] = "email is required"
    }
    if len(r.Password) < 8 {
        errs["password"] = "password must be at least 8 characters"
    }
    return errs
}

type LoginRequest struct {
    Email    string `json:"email"`
    Password string `json:"password"`
}

type AuthResponse struct {
    Token string `json:"token"`
    User  User   `json:"user"`
}
```

注意 `PasswordHash` 的 `json:"-"` 標籤——這確保密碼雜湊永遠不會被序列化到 JSON 回應中。

## 密碼雜湊

**永遠不要以明文儲存密碼。** 使用 bcrypt 進行雜湊：

```go
import "golang.org/x/crypto/bcrypt"

// 雜湊密碼
func hashPassword(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    return string(bytes), err
}

// 驗證密碼
func checkPassword(password, hash string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
    return err == nil
}
```

bcrypt 的特點：
- **自帶 salt**：每次雜湊結果都不同
- **可調整成本**：`DefaultCost` 是 10，數字越大越慢（也越安全）
- **抗暴力破解**：刻意設計得很慢

## 使用者 Store

### 資料庫 Migration

```go
func migrate(db *sql.DB) error {
    query := `
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT NOT NULL UNIQUE,
        email TEXT NOT NULL UNIQUE,
        password_hash TEXT NOT NULL,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    );

    CREATE TABLE IF NOT EXISTS books (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        author TEXT NOT NULL,
        user_id INTEGER REFERENCES users(id),
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    );
    `
    _, err := db.Exec(query)
    return err
}
```

### internal/store/user.go

```go
package store

import (
    "bookapi/internal/model"
    "database/sql"
    "errors"
    "fmt"

    "golang.org/x/crypto/bcrypt"
)

var ErrDuplicateEmail = errors.New("email already exists")
var ErrDuplicateUsername = errors.New("username already exists")

type UserStore struct {
    db *sql.DB
}

func NewUserStore(db *sql.DB) *UserStore {
    return &UserStore{db: db}
}

func (s *UserStore) Create(req model.RegisterRequest) (model.User, error) {
    // 雜湊密碼
    hash, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
    if err != nil {
        return model.User{}, fmt.Errorf("hashing password: %w", err)
    }

    result, err := s.db.Exec(
        "INSERT INTO users (username, email, password_hash) VALUES (?, ?, ?)",
        req.Username, req.Email, string(hash),
    )
    if err != nil {
        // 檢查 UNIQUE 約束違反
        if isUniqueViolation(err) {
            return model.User{}, ErrDuplicateEmail
        }
        return model.User{}, fmt.Errorf("creating user: %w", err)
    }

    id, _ := result.LastInsertId()
    return s.GetByID(int(id))
}

func (s *UserStore) GetByID(id int) (model.User, error) {
    var user model.User
    err := s.db.QueryRow(
        "SELECT id, username, email, password_hash, created_at FROM users WHERE id = ?",
        id,
    ).Scan(&user.ID, &user.Username, &user.Email, &user.PasswordHash, &user.CreatedAt)

    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return user, ErrNotFound
        }
        return user, fmt.Errorf("querying user: %w", err)
    }
    return user, nil
}

func (s *UserStore) GetByEmail(email string) (model.User, error) {
    var user model.User
    err := s.db.QueryRow(
        "SELECT id, username, email, password_hash, created_at FROM users WHERE email = ?",
        email,
    ).Scan(&user.ID, &user.Username, &user.Email, &user.PasswordHash, &user.CreatedAt)

    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return user, ErrNotFound
        }
        return user, fmt.Errorf("querying user by email: %w", err)
    }
    return user, nil
}

func (s *UserStore) Authenticate(email, password string) (model.User, error) {
    user, err := s.GetByEmail(email)
    if err != nil {
        return model.User{}, fmt.Errorf("invalid credentials")
    }

    err = bcrypt.CompareHashAndPassword([]byte(user.PasswordHash), []byte(password))
    if err != nil {
        return model.User{}, fmt.Errorf("invalid credentials")
    }

    return user, nil
}

func isUniqueViolation(err error) bool {
    return err != nil && (
        // SQLite unique constraint error
        contains(err.Error(), "UNIQUE constraint failed"))
}

func contains(s, substr string) bool {
    return len(s) >= len(substr) && searchString(s, substr)
}
```

注意 `Authenticate` 方法：無論是帳號不存在還是密碼錯誤，都回傳相同的錯誤訊息 "invalid credentials"。這可以防止攻擊者透過不同的錯誤訊息來探測哪些帳號存在。

## JWT 實作

### internal/auth/jwt.go

```go
package auth

import (
    "fmt"
    "time"

    "github.com/golang-jwt/jwt/v5"
)

type JWTManager struct {
    secretKey     []byte
    tokenDuration time.Duration
}

func NewJWTManager(secret string, duration time.Duration) *JWTManager {
    return &JWTManager{
        secretKey:     []byte(secret),
        tokenDuration: duration,
    }
}

type Claims struct {
    UserID int    `json:"user_id"`
    Email  string `json:"email"`
    jwt.RegisteredClaims
}

func (m *JWTManager) Generate(userID int, email string) (string, error) {
    claims := Claims{
        UserID: userID,
        Email:  email,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(m.tokenDuration)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(m.secretKey)
}

func (m *JWTManager) Verify(tokenString string) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(token *jwt.Token) (any, error) {
        // 確認簽名方法
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
        }
        return m.secretKey, nil
    })

    if err != nil {
        return nil, fmt.Errorf("parsing token: %w", err)
    }

    claims, ok := token.Claims.(*Claims)
    if !ok || !token.Valid {
        return nil, fmt.Errorf("invalid token")
    }

    return claims, nil
}
```

### JWT 的結構

JWT 由三部分組成：`Header.Payload.Signature`

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.    ← Header（演算法和類型）
eyJ1c2VyX2lkIjoxLCJlbWFpbCI6ImpAai5jb20ifQ.  ← Payload（資料）
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature（簽名）
```

## 認證中間件

### internal/middleware/auth.go

```go
package middleware

import (
    "bookapi/internal/auth"
    "context"
    "net/http"
    "strings"
)

type contextKey string

const UserIDKey contextKey = "userID"

func Auth(jwtManager *auth.JWTManager) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // 從 Authorization header 取得 token
            authHeader := r.Header.Get("Authorization")
            if authHeader == "" {
                http.Error(w, `{"error":"missing authorization header"}`, http.StatusUnauthorized)
                return
            }

            // 預期格式：Bearer <token>
            parts := strings.SplitN(authHeader, " ", 2)
            if len(parts) != 2 || parts[0] != "Bearer" {
                http.Error(w, `{"error":"invalid authorization format"}`, http.StatusUnauthorized)
                return
            }

            // 驗證 token
            claims, err := jwtManager.Verify(parts[1])
            if err != nil {
                http.Error(w, `{"error":"invalid or expired token"}`, http.StatusUnauthorized)
                return
            }

            // 將使用者 ID 存入 context
            ctx := context.WithValue(r.Context(), UserIDKey, claims.UserID)
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}

// 從 context 取得使用者 ID 的輔助函式
func GetUserID(ctx context.Context) (int, bool) {
    id, ok := ctx.Value(UserIDKey).(int)
    return id, ok
}
```

## 認證 Handler

### internal/handler/auth.go

```go
package handler

import (
    "bookapi/internal/auth"
    "bookapi/internal/model"
    "bookapi/internal/store"
    "encoding/json"
    "errors"
    "net/http"
)

type AuthHandler struct {
    userStore  *store.UserStore
    jwtManager *auth.JWTManager
}

func NewAuthHandler(us *store.UserStore, jm *auth.JWTManager) *AuthHandler {
    return &AuthHandler{userStore: us, jwtManager: jm}
}

func (h *AuthHandler) RegisterRoutes(mux *http.ServeMux) {
    mux.HandleFunc("POST /auth/register", h.Register)
    mux.HandleFunc("POST /auth/login", h.Login)
}

func (h *AuthHandler) Register(w http.ResponseWriter, r *http.Request) {
    var req model.RegisterRequest
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

    user, err := h.userStore.Create(req)
    if err != nil {
        if errors.Is(err, store.ErrDuplicateEmail) {
            writeError(w, http.StatusConflict, "Email already registered")
            return
        }
        writeError(w, http.StatusInternalServerError, "Failed to create user")
        return
    }

    // 產生 token
    token, err := h.jwtManager.Generate(user.ID, user.Email)
    if err != nil {
        writeError(w, http.StatusInternalServerError, "Failed to generate token")
        return
    }

    writeJSON(w, http.StatusCreated, model.AuthResponse{
        Token: token,
        User:  user,
    })
}

func (h *AuthHandler) Login(w http.ResponseWriter, r *http.Request) {
    var req model.LoginRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeError(w, http.StatusBadRequest, "Invalid JSON")
        return
    }

    user, err := h.userStore.Authenticate(req.Email, req.Password)
    if err != nil {
        writeError(w, http.StatusUnauthorized, "Invalid credentials")
        return
    }

    token, err := h.jwtManager.Generate(user.ID, user.Email)
    if err != nil {
        writeError(w, http.StatusInternalServerError, "Failed to generate token")
        return
    }

    writeJSON(w, http.StatusOK, model.AuthResponse{
        Token: token,
        User:  user,
    })
}
```

## 保護 API 端點

### 更新路由設定

```go
func main() {
    // ... 初始化 db, stores ...

    jwtManager := auth.NewJWTManager(
        os.Getenv("JWT_SECRET"),  // 從環境變數讀取
        24*time.Hour,              // Token 有效期
    )

    mux := http.NewServeMux()

    // 公開端點
    authHandler := handler.NewAuthHandler(userStore, jwtManager)
    authHandler.RegisterRoutes(mux)

    // 受保護的端點
    bookHandler := handler.NewBookHandler(bookStore)
    protectedMux := http.NewServeMux()
    bookHandler.RegisterRoutes(protectedMux)

    // 對 /books 路徑套用認證中間件
    mux.Handle("/books", middleware.Auth(jwtManager)(protectedMux))
    mux.Handle("/books/", middleware.Auth(jwtManager)(protectedMux))

    // 全域中間件
    var h http.Handler = mux
    h = middleware.Logging(h)
    h = middleware.Recovery(h)

    // ... 啟動伺服器 ...
}
```

### 在 Handler 中取得使用者身份

```go
func (h *BookHandler) Create(w http.ResponseWriter, r *http.Request) {
    // 取得當前登入的使用者 ID
    userID, ok := middleware.GetUserID(r.Context())
    if !ok {
        writeError(w, http.StatusUnauthorized, "User not found in context")
        return
    }

    // 建立書籍時關聯使用者
    // ...
}
```

## 測試認證流程

```bash
# 1. 註冊
curl -X POST http://localhost:8080/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"jack","email":"jack@example.com","password":"mypassword123"}'

# 回應：{"token":"eyJhbG...","user":{"id":1,"username":"jack",...}}

# 2. 登入
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"jack@example.com","password":"mypassword123"}'

# 3. 存取受保護的端點（帶 Token）
TOKEN="eyJhbG..."
curl http://localhost:8080/books \
  -H "Authorization: Bearer $TOKEN"

# 4. 不帶 Token → 401
curl http://localhost:8080/books
# {"error":"missing authorization header"}
```

## 安全性注意事項

### JWT Secret

```bash
# .env
JWT_SECRET=your-super-secret-key-at-least-32-chars-long
```

- Secret 必須足夠長且隨機
- 永遠不要硬編碼在程式碼中
- 永遠不要提交到版本控制

### Token 儲存

在前端：
- **建議**：存在 `httpOnly` cookie 中（防 XSS）
- **可接受**：存在 localStorage（簡單但有 XSS 風險）
- **不建議**：存在 URL 參數中

### 其他安全考量

- 使用 HTTPS（生產環境必須）
- 設定合理的 token 過期時間
- 考慮實作 refresh token 機制
- 限制登入嘗試次數（防暴力破解）
- 記錄認證失敗的日誌（偵測攻擊）

## 下一步

我們的 Web 應用現在有了完整的使用者系統。最後一篇文章，我們將學習如何為這個應用撰寫測試，並用 Docker 打包部署。
