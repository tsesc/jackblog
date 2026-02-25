+++
title = 'Go 語言系列（十五）：PostgreSQL 進階'
date = 2026-02-24T15:00:00+08:00
draft = false
tags = ['Golang', 'Go', 'PostgreSQL', 'pgx', 'Migration', '資料庫', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '從 SQLite 升級到 PostgreSQL：pgx driver、連線池設定、golang-migrate、Index 策略、Transaction Isolation 和 JSONB 操作'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第十五篇。前面我們用 SQLite 作為開發資料庫，現在升級到生產環境常用的 PostgreSQL，學習進階的資料庫功能。

## 用 Docker Compose 啟動 PostgreSQL

不需要在本地安裝 PostgreSQL，用 Docker 最方便：

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: bookstore
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

```bash
docker compose up -d
```

連線字串：`postgres://app:secret@localhost:5432/bookstore?sslmode=disable`

## pgx Driver

`pgx` 是目前 Go 生態中效能最好的 PostgreSQL driver，比傳統的 `lib/pq` 快且功能更完整。

安裝：

```bash
go get github.com/jackc/pgx/v5
go get github.com/jackc/pgx/v5/pgxpool
```

### 基本連線

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    ctx := context.Background()

    // pgxpool 自帶連線池
    pool, err := pgxpool.New(ctx, "postgres://app:secret@localhost:5432/bookstore?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer pool.Close()

    // 測試連線
    if err := pool.Ping(ctx); err != nil {
        log.Fatal(err)
    }
    fmt.Println("Connected to PostgreSQL!")
}
```

### 搭配 database/sql 使用

如果你想保持用 `database/sql` 的介面（例如搭配 sqlx）：

```go
import (
    "database/sql"
    _ "github.com/jackc/pgx/v5/stdlib"
)

db, err := sql.Open("pgx", "postgres://app:secret@localhost:5432/bookstore")
```

## 連線池設定

生產環境的連線池設定非常重要：

```go
config, err := pgxpool.ParseConfig("postgres://app:secret@localhost:5432/bookstore")
if err != nil {
    log.Fatal(err)
}

// 連線池參數
config.MaxConns = 25              // 最大連線數
config.MinConns = 5               // 最小維持的空閒連線
config.MaxConnLifetime = time.Hour // 每條連線最長存活時間
config.MaxConnIdleTime = 30 * time.Minute // 空閒連線最長存活時間
config.HealthCheckPeriod = time.Minute    // 健康檢查間隔

pool, err := pgxpool.NewWithConfig(context.Background(), config)
```

**設定建議：**

| 參數 | 建議值 | 說明 |
|------|--------|------|
| MaxConns | CPU 核心數 * 2 + 磁碟數 | 經驗法則，通常 20-50 |
| MinConns | MaxConns / 5 | 維持最少的熱連線 |
| MaxConnLifetime | 1h | 避免長時間持有連線 |
| MaxConnIdleTime | 30m | 回收長時間沒用的連線 |

## Database Migration

`golang-migrate` 是 Go 生態中最廣泛使用的 migration 工具。

### 安裝

```bash
# CLI 工具
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
```

### 建立 Migration

```bash
# 建立 migration 檔案（自動加上時間戳）
migrate create -ext sql -dir migrations -seq create_users
```

這會產生兩個檔案：

```sql
-- migrations/000001_create_users.up.sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
```

```sql
-- migrations/000001_create_users.down.sql
DROP TABLE IF EXISTS users;
```

### 執行與回滾

```bash
# 執行所有 pending migration
migrate -path migrations -database "postgres://app:secret@localhost:5432/bookstore?sslmode=disable" up

# 回滾上一個 migration
migrate -path migrations -database "..." down 1

# 回滾所有
migrate -path migrations -database "..." down

# 查看目前版本
migrate -path migrations -database "..." version
```

### 在 Go 程式碼中整合

```go
import (
    "github.com/golang-migrate/migrate/v4"
    _ "github.com/golang-migrate/migrate/v4/database/postgres"
    _ "github.com/golang-migrate/migrate/v4/source/file"
)

func runMigrations(dbURL string) error {
    m, err := migrate.New("file://migrations", dbURL)
    if err != nil {
        return err
    }

    if err := m.Up(); err != nil && err != migrate.ErrNoChange {
        return err
    }
    return nil
}
```

## PostgreSQL 進階功能

### Index 策略

```sql
-- B-tree（預設）：適用於等值和範圍查詢
CREATE INDEX idx_books_title ON books(title);

-- 複合 Index：查詢條件經常一起出現時
CREATE INDEX idx_books_author_year ON books(author, year);

-- 部分 Index：只對符合條件的行建立索引
CREATE INDEX idx_active_users ON users(email) WHERE deleted_at IS NULL;

-- GIN Index：適用於全文搜尋和 JSONB
CREATE INDEX idx_books_metadata ON books USING GIN(metadata);
```

在 Go 中檢查查詢是否使用 index：

```go
var plan string
err := pool.QueryRow(ctx, "EXPLAIN ANALYZE SELECT * FROM users WHERE email = $1", "jack@example.com").Scan(&plan)
fmt.Println(plan) // 看到 "Index Scan" 就表示有用到 index
```

### Transaction Isolation Level

```go
ctx := context.Background()

// 開始一個帶有隔離等級的 transaction
tx, err := pool.BeginTx(ctx, pgx.TxOptions{
    IsoLevel: pgx.ReadCommitted, // 預設
})
// 其他選項：
// pgx.RepeatableRead  — 避免不可重複讀
// pgx.Serializable    — 最嚴格，完全序列化

defer tx.Rollback(ctx)

// 執行操作
_, err = tx.Exec(ctx, "UPDATE accounts SET balance = balance - $1 WHERE id = $2", 100, 1)
if err != nil {
    return err
}
_, err = tx.Exec(ctx, "UPDATE accounts SET balance = balance + $1 WHERE id = $2", 100, 2)
if err != nil {
    return err
}

return tx.Commit(ctx)
```

### JSONB 欄位

PostgreSQL 的 JSONB 讓你在關聯式資料庫中使用半結構化資料：

```sql
-- 建立帶有 JSONB 欄位的表
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    metadata JSONB DEFAULT '{}'
);

-- 插入 JSONB 資料
INSERT INTO products (name, metadata)
VALUES ('Laptop', '{"brand": "Apple", "specs": {"ram": 16, "storage": 512}}');
```

在 Go 中操作：

```go
import "encoding/json"

type ProductMetadata struct {
    Brand string `json:"brand"`
    Specs struct {
        RAM     int `json:"ram"`
        Storage int `json:"storage"`
    } `json:"specs"`
}

// 寫入
meta := ProductMetadata{Brand: "Apple"}
meta.Specs.RAM = 16
meta.Specs.Storage = 512

metaJSON, _ := json.Marshal(meta)
_, err := pool.Exec(ctx,
    "INSERT INTO products (name, metadata) VALUES ($1, $2)",
    "Laptop", metaJSON,
)

// 讀取
var name string
var metadata ProductMetadata
err := pool.QueryRow(ctx,
    "SELECT name, metadata FROM products WHERE id = $1", 1,
).Scan(&name, &metadata)

// JSONB 查詢（用 PostgreSQL 的 @> 運算子）
rows, err := pool.Query(ctx,
    `SELECT name FROM products WHERE metadata @> '{"brand": "Apple"}'`,
)

// JSONB 取出特定欄位
var brand string
err := pool.QueryRow(ctx,
    "SELECT metadata->>'brand' FROM products WHERE id = $1", 1,
).Scan(&brand)
```

## 下一步

資料庫的基礎穩固了，下一篇我們來學 Go 的另一個強項——gRPC，了解如何用 Protocol Buffers 定義服務介面，實現高效能的 RPC 通訊。
