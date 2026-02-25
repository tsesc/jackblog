+++
title = 'Go 語言系列（十四）：ORM 與資料庫工具 — GORM vs sqlx'
date = 2026-02-24T14:00:00+08:00
draft = false
tags = ['Golang', 'Go', 'GORM', 'sqlx', 'ORM', '資料庫', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '對比 Go 的兩大資料庫工具：GORM（全功能 ORM）和 sqlx（SQL-first 擴展），從 Model 定義、CRUD、關聯查詢到效能，幫助你選擇適合的工具'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第十四篇。前面我們用 `database/sql` 手寫 SQL 操作資料庫，現在來看看 ORM 和資料庫工具如何簡化這些工作。

## ORM vs 原生 SQL

| 面向 | 原生 SQL (database/sql) | ORM (GORM) | SQL-first (sqlx) |
|------|------------------------|-------------|-------------------|
| 學習曲線 | 需要懂 SQL | 可以少寫 SQL | 需要懂 SQL |
| 開發速度 | 較慢（模板程式碼多） | 快（自動產生 SQL） | 中等 |
| SQL 控制力 | 完全控制 | 有限（複雜查詢困難） | 完全控制 |
| 效能 | 最佳 | 有反射開銷 | 接近原生 |
| Migration | 需自行處理 | 內建 AutoMigrate | 需自行處理 |

## GORM

安裝：

```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/sqlite  # 或 postgres、mysql
```

### Model 定義

```go
package main

import (
    "time"
    "gorm.io/gorm"
)

type User struct {
    ID        uint           `gorm:"primaryKey" json:"id"`
    Name      string         `gorm:"size:100;not null" json:"name"`
    Email     string         `gorm:"uniqueIndex;not null" json:"email"`
    Books     []Book         `gorm:"foreignKey:UserID" json:"books"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"` // 軟刪除
}

type Book struct {
    ID     uint   `gorm:"primaryKey" json:"id"`
    Title  string `gorm:"size:200;not null" json:"title"`
    Author string `gorm:"size:100" json:"author"`
    UserID uint   `json:"user_id"`
}
```

### 連線與 AutoMigrate

```go
import (
    "gorm.io/driver/sqlite"
    "gorm.io/gorm"
)

func main() {
    db, err := gorm.Open(sqlite.Open("app.db"), &gorm.Config{})
    if err != nil {
        panic("failed to connect database")
    }

    // AutoMigrate 自動建立或更新表結構
    db.AutoMigrate(&User{}, &Book{})
}
```

### CRUD 操作

```go
// Create
user := User{Name: "Jack", Email: "jack@example.com"}
result := db.Create(&user)
// result.RowsAffected = 1
// user.ID 自動填入

// Read
var u User
db.First(&u, 1)                          // 找 ID=1
db.Where("email = ?", "jack@example.com").First(&u) // 條件查詢

// 查詢多筆
var users []User
db.Where("name LIKE ?", "%Jack%").Find(&users)

// Update
db.Model(&user).Update("name", "Jack Tse")
db.Model(&user).Updates(User{Name: "Jack Tse", Email: "new@example.com"})

// Delete（軟刪除，因為 User 有 DeletedAt 欄位）
db.Delete(&user, 1)

// 真正刪除
db.Unscoped().Delete(&user, 1)
```

### 關聯查詢

```go
// 建立使用者和書籍
db.Create(&User{
    Name:  "Jack",
    Email: "jack@example.com",
    Books: []Book{
        {Title: "Go in Action", Author: "William Kennedy"},
        {Title: "The Go Programming Language", Author: "Donovan & Kernighan"},
    },
})

// Preload 關聯資料
var user User
db.Preload("Books").First(&user, 1)
// user.Books 會自動填入

// 條件 Preload
db.Preload("Books", "author LIKE ?", "%Kennedy%").First(&user, 1)
```

### Hook

```go
// 在建立之前自動處理
func (u *User) BeforeCreate(tx *gorm.DB) error {
    if u.Email == "" {
        return errors.New("email is required")
    }
    return nil
}

// 在刪除之後清理關聯
func (u *User) AfterDelete(tx *gorm.DB) error {
    tx.Where("user_id = ?", u.ID).Delete(&Book{})
    return nil
}
```

### Transaction

```go
err := db.Transaction(func(tx *gorm.DB) error {
    user := User{Name: "Jack", Email: "jack@example.com"}
    if err := tx.Create(&user).Error; err != nil {
        return err // 回傳 error 會自動 rollback
    }

    book := Book{Title: "New Book", UserID: user.ID}
    if err := tx.Create(&book).Error; err != nil {
        return err
    }

    return nil // 回傳 nil 會自動 commit
})
```

## sqlx

安裝：

```bash
go get -u github.com/jmoiron/sqlx
go get -u github.com/mattn/go-sqlite3  # 或 pgx
```

### 連線

```go
import (
    "github.com/jmoiron/sqlx"
    _ "github.com/mattn/go-sqlite3"
)

func main() {
    db, err := sqlx.Connect("sqlite3", "app.db")
    if err != nil {
        panic(err)
    }
    defer db.Close()

    // 手動建表
    db.MustExec(`
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT UNIQUE NOT NULL,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        )
    `)
}
```

### Model 定義

```go
type User struct {
    ID        int       `db:"id" json:"id"`
    Name      string    `db:"name" json:"name"`
    Email     string    `db:"email" json:"email"`
    CreatedAt time.Time `db:"created_at" json:"created_at"`
}

type Book struct {
    ID     int    `db:"id" json:"id"`
    Title  string `db:"title" json:"title"`
    Author string `db:"author" json:"author"`
    UserID int    `db:"user_id" json:"user_id"`
}
```

### CRUD 操作

```go
// Create
result, err := db.NamedExec(
    `INSERT INTO users (name, email) VALUES (:name, :email)`,
    &User{Name: "Jack", Email: "jack@example.com"},
)
id, _ := result.LastInsertId()

// Read：單筆
var user User
err = db.Get(&user, "SELECT * FROM users WHERE id = ?", 1)

// Read：多筆
var users []User
err = db.Select(&users, "SELECT * FROM users WHERE name LIKE ?", "%Jack%")

// Update
_, err = db.NamedExec(
    `UPDATE users SET name = :name WHERE id = :id`,
    &User{ID: 1, Name: "Jack Tse"},
)

// Delete
_, err = db.Exec("DELETE FROM users WHERE id = ?", 1)
```

### NamedQuery

```go
// 具名參數查詢
rows, err := db.NamedQuery(
    `SELECT * FROM users WHERE name = :name AND email = :email`,
    map[string]interface{}{
        "name":  "Jack",
        "email": "jack@example.com",
    },
)

for rows.Next() {
    var user User
    rows.StructScan(&user)
}
```

### In 語法

```go
// IN 查詢
ids := []int{1, 2, 3}
query, args, err := sqlx.In("SELECT * FROM users WHERE id IN (?)", ids)
query = db.Rebind(query) // 轉換佔位符格式

var users []User
err = db.Select(&users, query, args...)
```

### Transaction

```go
tx, err := db.Beginx()
if err != nil {
    return err
}

_, err = tx.NamedExec(
    `INSERT INTO users (name, email) VALUES (:name, :email)`,
    &User{Name: "Jack", Email: "jack@example.com"},
)
if err != nil {
    tx.Rollback()
    return err
}

_, err = tx.Exec(
    `INSERT INTO books (title, user_id) VALUES (?, ?)`,
    "New Book", 1,
)
if err != nil {
    tx.Rollback()
    return err
}

return tx.Commit()
```

## GORM vs sqlx 對比

| 維度 | GORM | sqlx |
|------|------|------|
| **抽象程度** | 高（不需要寫 SQL） | 低（你寫 SQL，它幫你掃描） |
| **學習曲線** | 需要學 GORM API | 會 SQL 就能用 |
| **CRUD 速度** | 很快（一行搞定） | 中等（要寫 SQL） |
| **複雜查詢** | 困難（易掉入 N+1） | 簡單（就是寫 SQL） |
| **效能** | 有反射開銷 | 接近原生 database/sql |
| **Migration** | 內建 AutoMigrate | 需搭配其他工具 |
| **關聯查詢** | 內建 Preload | 自行 JOIN |
| **Debug** | 需要看產生的 SQL | 所見即所得 |

### 如何選擇？

**選 GORM 如果：**
- 快速開發 CRUD 為主的應用
- 需要 AutoMigrate 和軟刪除等功能
- 團隊 SQL 經驗較少

**選 sqlx 如果：**
- 需要精確控制 SQL
- 效能敏感的場景
- 複雜的查詢需求（多表 JOIN、子查詢）
- 偏好「SQL 是一等公民」的哲學

**混合使用也是一種策略：** 用 GORM 做快速的 CRUD，用 sqlx 處理複雜的報表查詢。

## 下一步

有了 ORM 工具，下一步要升級資料庫。下一篇我們從 SQLite 切換到 PostgreSQL，學習連線池、Migration、Index 和 JSONB 等進階功能。
