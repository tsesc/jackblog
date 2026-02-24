+++
title = 'Go 語言系列（八）：資料庫整合'
date = 2026-03-10T10:00:00+08:00
draft = false
tags = ['Golang', 'Go', 'SQLite', 'database/sql', '資料庫', 'CRUD', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '學習在 Go Web 應用中整合 SQLite 資料庫：database/sql 標準介面、Migration、CRUD 操作與 Repository Pattern'
toc = true
weight = 3
+++

> 這是 **Go 語言從零到 Web 應用**系列的第八篇。上一篇我們建立了結構清晰的專案，但資料存在記憶體中。這篇要把資料存進真正的資料庫。

## 為什麼選 SQLite？

對於教學和小型應用來說，SQLite 是完美的選擇：

- **零配置**：不需要安裝資料庫伺服器
- **單一檔案**：整個資料庫就是一個檔案
- **效能優秀**：對於大多數應用場景都足夠快
- **SQL 相容**：學到的 SQL 知識可以直接用在 PostgreSQL、MySQL

而且 Go 的 `database/sql` 是統一的資料庫介面，只要換一個 driver 就能切換到其他資料庫。

## database/sql 介紹

`database/sql` 是 Go 標準庫提供的資料庫抽象層。它定義了統一的介面，不同的資料庫只需要提供對應的 driver。

常見的 driver：
- **SQLite**: `github.com/mattn/go-sqlite3`（CGO）或 `modernc.org/sqlite`（純 Go）
- **PostgreSQL**: `github.com/lib/pq` 或 `github.com/jackc/pgx`
- **MySQL**: `github.com/go-sql-driver/mysql`

我們使用純 Go 實作的 SQLite driver，不需要 CGO：

```bash
go get modernc.org/sqlite
```

## 連接資料庫

```go
import (
    "database/sql"
    _ "modernc.org/sqlite"  // 註冊 SQLite driver
)

func main() {
    db, err := sql.Open("sqlite", "./bookapi.db")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // 驗證連線
    if err := db.Ping(); err != nil {
        log.Fatal(err)
    }

    fmt.Println("Database connected!")
}
```

注意 `_ "modernc.org/sqlite"` 這行——前綴的 `_` 表示只引入套件的副作用（註冊 driver），不直接使用它的任何匯出名稱。

### 連線池設定

`sql.DB` 不是單一連線，而是一個連線池。可以調整池的參數：

```go
db.SetMaxOpenConns(25)                  // 最大開啟連線數
db.SetMaxIdleConns(5)                   // 最大閒置連線數
db.SetConnMaxLifetime(5 * time.Minute)  // 連線最大存活時間
```

## Migration（資料庫遷移）

Migration 是管理資料庫 schema 變更的機制。

### 簡單的 Migration 實作

```go
func migrate(db *sql.DB) error {
    query := `
    CREATE TABLE IF NOT EXISTS books (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        author TEXT NOT NULL,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    );

    CREATE INDEX IF NOT EXISTS idx_books_author ON books(author);
    `

    _, err := db.Exec(query)
    return err
}
```

在 `main` 中啟動時執行：

```go
if err := migrate(db); err != nil {
    log.Fatalf("Migration failed: %v", err)
}
```

對於更複雜的專案，可以使用專門的 migration 工具如 `golang-migrate/migrate`，但在教學階段，簡單的 `CREATE TABLE IF NOT EXISTS` 就夠用了。

## CRUD 操作

### Create（新增）

```go
func createBook(db *sql.DB, title, author string) (int64, error) {
    result, err := db.Exec(
        "INSERT INTO books (title, author) VALUES (?, ?)",
        title, author,
    )
    if err != nil {
        return 0, fmt.Errorf("creating book: %w", err)
    }

    id, err := result.LastInsertId()
    if err != nil {
        return 0, fmt.Errorf("getting last insert id: %w", err)
    }

    return id, nil
}
```

### Read（讀取）

**讀取單筆：**

```go
func getBook(db *sql.DB, id int) (model.Book, error) {
    var book model.Book

    err := db.QueryRow(
        "SELECT id, title, author, created_at, updated_at FROM books WHERE id = ?",
        id,
    ).Scan(&book.ID, &book.Title, &book.Author, &book.CreatedAt, &book.UpdatedAt)

    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return book, ErrNotFound
        }
        return book, fmt.Errorf("querying book %d: %w", id, err)
    }

    return book, nil
}
```

`QueryRow` 回傳最多一行結果。如果沒有匹配的資料，`Scan` 會回傳 `sql.ErrNoRows`。

**讀取多筆：**

```go
func listBooks(db *sql.DB) ([]model.Book, error) {
    rows, err := db.Query(
        "SELECT id, title, author, created_at, updated_at FROM books ORDER BY id DESC",
    )
    if err != nil {
        return nil, fmt.Errorf("querying books: %w", err)
    }
    defer rows.Close()

    var books []model.Book
    for rows.Next() {
        var b model.Book
        if err := rows.Scan(&b.ID, &b.Title, &b.Author, &b.CreatedAt, &b.UpdatedAt); err != nil {
            return nil, fmt.Errorf("scanning book: %w", err)
        }
        books = append(books, b)
    }

    // 檢查迭代過程中是否有錯誤
    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("iterating rows: %w", err)
    }

    return books, nil
}
```

重要：**永遠要 `defer rows.Close()`**，否則會造成連線洩漏。

### Update（更新）

```go
func updateBook(db *sql.DB, id int, title, author string) error {
    result, err := db.Exec(
        "UPDATE books SET title = ?, author = ?, updated_at = CURRENT_TIMESTAMP WHERE id = ?",
        title, author, id,
    )
    if err != nil {
        return fmt.Errorf("updating book %d: %w", id, err)
    }

    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return fmt.Errorf("getting rows affected: %w", err)
    }

    if rowsAffected == 0 {
        return ErrNotFound
    }

    return nil
}
```

### Delete（刪除）

```go
func deleteBook(db *sql.DB, id int) error {
    result, err := db.Exec("DELETE FROM books WHERE id = ?", id)
    if err != nil {
        return fmt.Errorf("deleting book %d: %w", id, err)
    }

    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return fmt.Errorf("getting rows affected: %w", err)
    }

    if rowsAffected == 0 {
        return ErrNotFound
    }

    return nil
}
```

## SQL Injection 防護

**永遠使用參數化查詢，不要拼接 SQL 字串：**

```go
// 危險！SQL Injection 漏洞
query := "SELECT * FROM books WHERE title = '" + userInput + "'"

// 安全！使用參數化查詢
db.Query("SELECT * FROM books WHERE title = ?", userInput)
```

`?` 佔位符會被 driver 正確地 escape，防止 SQL Injection。

不同資料庫的佔位符格式：
- **SQLite / MySQL**: `?`
- **PostgreSQL**: `$1`, `$2`, `$3`

## Repository Pattern

將資料庫操作封裝成 Repository，讓 handler 不直接碰 SQL：

### internal/store/book.go（改用資料庫版本）

```go
package store

import (
    "bookapi/internal/model"
    "database/sql"
    "errors"
    "fmt"
)

var ErrNotFound = errors.New("not found")

type BookStore struct {
    db *sql.DB
}

func NewBookStore(db *sql.DB) *BookStore {
    return &BookStore{db: db}
}

func (s *BookStore) List() ([]model.Book, error) {
    rows, err := s.db.Query(
        "SELECT id, title, author, created_at, updated_at FROM books ORDER BY id DESC",
    )
    if err != nil {
        return nil, fmt.Errorf("querying books: %w", err)
    }
    defer rows.Close()

    var books []model.Book
    for rows.Next() {
        var b model.Book
        if err := rows.Scan(&b.ID, &b.Title, &b.Author, &b.CreatedAt, &b.UpdatedAt); err != nil {
            return nil, fmt.Errorf("scanning book: %w", err)
        }
        books = append(books, b)
    }

    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("iterating rows: %w", err)
    }

    return books, nil
}

func (s *BookStore) GetByID(id int) (model.Book, error) {
    var book model.Book
    err := s.db.QueryRow(
        "SELECT id, title, author, created_at, updated_at FROM books WHERE id = ?",
        id,
    ).Scan(&book.ID, &book.Title, &book.Author, &book.CreatedAt, &book.UpdatedAt)

    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return book, ErrNotFound
        }
        return book, fmt.Errorf("querying book %d: %w", id, err)
    }

    return book, nil
}

func (s *BookStore) Create(req model.CreateBookRequest) (model.Book, error) {
    result, err := s.db.Exec(
        "INSERT INTO books (title, author) VALUES (?, ?)",
        req.Title, req.Author,
    )
    if err != nil {
        return model.Book{}, fmt.Errorf("inserting book: %w", err)
    }

    id, err := result.LastInsertId()
    if err != nil {
        return model.Book{}, fmt.Errorf("getting last insert id: %w", err)
    }

    return s.GetByID(int(id))
}

func (s *BookStore) Update(id int, req model.UpdateBookRequest) (model.Book, error) {
    book, err := s.GetByID(id)
    if err != nil {
        return book, err
    }

    if req.Title != nil {
        book.Title = *req.Title
    }
    if req.Author != nil {
        book.Author = *req.Author
    }

    _, err = s.db.Exec(
        "UPDATE books SET title = ?, author = ?, updated_at = CURRENT_TIMESTAMP WHERE id = ?",
        book.Title, book.Author, id,
    )
    if err != nil {
        return model.Book{}, fmt.Errorf("updating book %d: %w", id, err)
    }

    return s.GetByID(id)
}

func (s *BookStore) Delete(id int) error {
    result, err := s.db.Exec("DELETE FROM books WHERE id = ?", id)
    if err != nil {
        return fmt.Errorf("deleting book %d: %w", id, err)
    }

    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return fmt.Errorf("getting rows affected: %w", err)
    }

    if rowsAffected == 0 {
        return ErrNotFound
    }

    return nil
}
```

### 更新 main.go

```go
func main() {
    // 連接資料庫
    db, err := sql.Open("sqlite", "./bookapi.db")
    if err != nil {
        log.Fatalf("Opening database: %v", err)
    }
    defer db.Close()

    // 執行 migration
    if err := migrate(db); err != nil {
        log.Fatalf("Migration: %v", err)
    }

    // 初始化 store（現在接受 *sql.DB）
    bookStore := store.NewBookStore(db)

    // 其餘和之前一樣...
}
```

Handler 層的程式碼完全不需要改動——只是 store 內部從記憶體換成了資料庫。這就是 Repository Pattern 的好處。

## 交易（Transactions）

當多個操作需要原子性執行時，使用 transaction：

```go
func (s *BookStore) TransferBooks(fromAuthor, toAuthor string) error {
    tx, err := s.db.Begin()
    if err != nil {
        return fmt.Errorf("beginning transaction: %w", err)
    }
    // 確保 transaction 最終被 commit 或 rollback
    defer tx.Rollback()

    // 在 transaction 中執行操作
    _, err = tx.Exec(
        "UPDATE books SET author = ? WHERE author = ?",
        toAuthor, fromAuthor,
    )
    if err != nil {
        return fmt.Errorf("updating author: %w", err)
    }

    // 提交 transaction
    if err := tx.Commit(); err != nil {
        return fmt.Errorf("committing transaction: %w", err)
    }

    return nil
}
```

`defer tx.Rollback()` 是安全的——如果已經 Commit 成功，Rollback 會是 no-op。

## Context 與資料庫

在 Web 應用中，每個請求都有 context，應該傳遞給資料庫操作：

```go
func (s *BookStore) GetByID(ctx context.Context, id int) (model.Book, error) {
    var book model.Book
    err := s.db.QueryRowContext(ctx,
        "SELECT id, title, author, created_at, updated_at FROM books WHERE id = ?",
        id,
    ).Scan(&book.ID, &book.Title, &book.Author, &book.CreatedAt, &book.UpdatedAt)

    // ...
}

// Handler 中傳遞 context
func (h *BookHandler) Get(w http.ResponseWriter, r *http.Request) {
    // r.Context() 在請求取消時會自動 cancel
    book, err := h.store.GetByID(r.Context(), id)
    // ...
}
```

使用 `QueryRowContext`、`QueryContext`、`ExecContext` 而非不帶 context 的版本。這樣當客戶端斷線時，長時間運行的查詢會被自動取消。

## 常見陷阱

### 忘記關閉 Rows

```go
// 錯誤：沒有 Close
rows, _ := db.Query("SELECT ...")
for rows.Next() { ... }

// 正確：用 defer Close
rows, err := db.Query("SELECT ...")
if err != nil { return err }
defer rows.Close()
```

### Scan 的型別不匹配

```go
// 資料庫欄位可能是 NULL
var name sql.NullString
row.Scan(&name)

if name.Valid {
    fmt.Println(name.String)
}
```

### 在迴圈中開啟 Rows 沒關閉

```go
// 危險：每次迴圈都開一個 rows 但沒關
for _, id := range ids {
    rows, _ := db.Query("SELECT ...", id)
    // 用 QueryRow 替代，或確保每次都 Close
}

// 更好：用 QueryRow 或批次查詢
for _, id := range ids {
    var book model.Book
    db.QueryRow("SELECT ... WHERE id = ?", id).Scan(...)
}
```

## 下一步

我們的 Web 應用現在有了持久化儲存。下一篇要加入使用者認證——讓 API 能識別「誰」在操作，並保護需要登入才能存取的端點。
