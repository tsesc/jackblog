+++
title = 'MySQL 索引最佳實踐完整指南'
date = 2025-08-25T09:00:00+08:00
draft = false
tags = ['MySQL', '資料庫', '索引優化', '效能調校', 'SQL']
categories = ['技術筆記']
author = 'Jack'
description = '深入解析 MySQL 索引的設計原則、最佳實踐和常見錯誤。從基礎概念到進階技巧，幫助你優化資料庫查詢效能。'
toc = true
weight = 1
+++

在資料庫優化的世界裡，索引是提升查詢效能的關鍵武器。本文基於 Rick James 的 [MySQL Indexing Cookbook](https://mysql.rjweb.org/doc.php/index_cookbook_mysql) 整理出 MySQL 索引設計的核心原則與實戰技巧。

## 為什麼索引如此重要？

想像一下在沒有目錄的百科全書中查找特定資訊 - 這就是沒有索引的資料表查詢。適當的索引可以將查詢時間從數秒縮短到毫秒級別，特別是在處理大量資料時。

## 索引設計的黃金法則

### 1. 三步驟索引設計演算法

建立有效索引的基本步驟：

**步驟 1：優先處理等值條件**
將 WHERE 子句中與常數比較的欄位放在索引最前面：

```sql
-- 查詢
SELECT * FROM users
WHERE status = 'active' AND department = 'IT';

-- 索引設計
INDEX(status, department)
```

**步驟 2：加入範圍條件欄位**
範圍查詢（BETWEEN、>、< 等）的欄位放在等值條件之後：

```sql
-- 查詢
SELECT * FROM orders
WHERE status = 'pending'
  AND created_date >= '2025-01-01';

-- 索引設計
INDEX(status, created_date)  -- 等值在前，範圍在後
```

**步驟 3：考慮 GROUP BY 和 ORDER BY**
如果沒有範圍條件，可以將 GROUP BY 或 ORDER BY 的欄位加入索引：

```sql
-- 查詢
SELECT department, COUNT(*)
FROM employees
WHERE company_id = 100
GROUP BY department;

-- 索引設計
INDEX(company_id, department)
```

### 2. 複合索引的藝術

複合索引（Composite Index）是包含多個欄位的索引，其設計原則：

#### 欄位順序至關重要

```sql
-- 這兩個索引是不同的！
INDEX(last_name, first_name)   -- 適合 WHERE last_name = 'Chen'
INDEX(first_name, last_name)   -- 適合 WHERE first_name = 'Jack'
```

#### 最左前綴原則

複合索引可以當作多個索引使用：

```sql
-- 擁有索引
INDEX(a, b, c)

-- 相當於同時擁有
INDEX(a)
INDEX(a, b)
INDEX(a, b, c)
```

### 3. 覆蓋索引（Covering Index）

當索引包含查詢所需的所有欄位時，MySQL 可以直接從索引取得資料，無需回表：

```sql
-- 查詢
SELECT user_id, username, email
FROM users
WHERE status = 'active';

-- 覆蓋索引
INDEX(status, user_id, username, email)
```

**優點：**
- 大幅減少磁碟 I/O
- 查詢速度提升數倍

**限制：**
- 建議不超過 5 個欄位
- 需要權衡索引大小與效能提升

## 實戰案例分析

### 案例 1：多對多關聯表優化

```sql
-- 文章標籤關聯表
CREATE TABLE article_tags (
    article_id INT,
    tag_id INT,
    PRIMARY KEY(article_id, tag_id),  -- 複合主鍵
    INDEX(tag_id, article_id)          -- 反向索引
);

-- 兩個索引支援雙向查詢
-- 1. 查詢文章的所有標籤：使用 PRIMARY KEY
-- 2. 查詢標籤的所有文章：使用 INDEX(tag_id, article_id)
```

### 案例 2：時間範圍查詢優化

```sql
-- 低效查詢
SELECT * FROM logs
WHERE created_at BETWEEN '2025-01-01' AND '2025-01-31'
  AND level = 'ERROR';

-- 原索引（效能差）
INDEX(created_at, level)

-- 優化後索引（效能好）
INDEX(level, created_at)
-- 因為 level 是等值條件，應該放在範圍條件 created_at 之前
```

### 案例 3：分頁查詢優化

```sql
-- 問題查詢：深度分頁
SELECT * FROM products
ORDER BY created_at DESC
LIMIT 10000, 20;  -- 跳過 10000 筆資料

-- 優化方案：使用覆蓋索引 + 子查詢
SELECT p.* FROM products p
INNER JOIN (
    SELECT id FROM products
    ORDER BY created_at DESC
    LIMIT 10000, 20
) AS tmp ON p.id = tmp.id;

-- 需要索引
INDEX(created_at, id)  -- 覆蓋索引
```

## 常見錯誤與陷阱

### 1. 低基數索引陷阱

```sql
-- 錯誤：為布林欄位建立單獨索引
INDEX(is_deleted)  -- 只有 true/false 兩個值

-- 正確：結合其他欄位建立複合索引
INDEX(is_deleted, created_at)
```

**原因：** MySQL 優化器對於選擇性低的索引（如性別、狀態等）通常會選擇全表掃描。

### 2. 索引過多的問題

```sql
-- 錯誤：為每個欄位都建立索引
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100),
    phone VARCHAR(20),
    address TEXT,
    INDEX(username),
    INDEX(email),
    INDEX(phone),
    INDEX(address(100))  -- 更糟糕的是索引 TEXT 欄位
);
```

**問題：**
- 降低 INSERT/UPDATE/DELETE 效能
- 增加儲存空間
- 維護成本高

**建議：** 每個表保持在 6 個索引以內

### 3. 索引失效的情況

```sql
-- 索引失效案例 1：函數操作
WHERE YEAR(created_at) = 2025  -- 索引失效
-- 改為
WHERE created_at >= '2025-01-01' AND created_at < '2026-01-01'

-- 索引失效案例 2：隱式類型轉換
WHERE phone = 123456789  -- phone 是 VARCHAR，索引失效
-- 改為
WHERE phone = '123456789'

-- 索引失效案例 3：LIKE 前綴匹配
WHERE username LIKE '%jack%'  -- 索引失效
-- 如果可能，改為
WHERE username LIKE 'jack%'  -- 可以使用索引
```

## 進階優化技巧

### 1. 前綴索引（Prefix Index）

對於長字串欄位，可以只索引前 N 個字元：

```sql
-- 原始欄位很長
ALTER TABLE articles ADD INDEX(title(50));  -- 只索引前 50 個字元

-- 選擇合適的前綴長度
SELECT
    COUNT(DISTINCT LEFT(title, 10)) / COUNT(DISTINCT title) AS sel10,
    COUNT(DISTINCT LEFT(title, 20)) / COUNT(DISTINCT title) AS sel20,
    COUNT(DISTINCT LEFT(title, 50)) / COUNT(DISTINCT title) AS sel50
FROM articles;
-- 選擇接近 1.0 的最小長度
```

### 2. 索引合併（Index Merge）

MySQL 可以同時使用多個索引：

```sql
-- 查詢
SELECT * FROM users
WHERE username = 'jack' OR email = 'jack@example.com';

-- 需要的索引
INDEX(username)
INDEX(email)
-- MySQL 會使用 Index Merge 優化
```

### 3. 使用 EXPLAIN 分析查詢

```sql
EXPLAIN SELECT * FROM users WHERE status = 'active';
```

重點關注：
- `type`：最好是 const、eq_ref、ref
- `key`：實際使用的索引
- `rows`：掃描的行數（越少越好）
- `Extra`：Using index（覆蓋索引）是好訊號

## 索引維護最佳實踐

### 1. 定期分析索引使用情況

```sql
-- 查看未使用的索引
SELECT * FROM sys.schema_unused_indexes;

-- 查看重複索引
SELECT * FROM sys.schema_redundant_indexes;
```

### 2. 監控慢查詢

```sql
-- 啟用慢查詢日誌
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;  -- 2 秒以上的查詢
```

### 3. 定期優化表

```sql
-- 重建索引，回收空間
OPTIMIZE TABLE users;

-- 更新統計資訊
ANALYZE TABLE users;
```

## 總結與檢查清單

設計索引時的檢查清單：

- [ ] WHERE 條件的欄位都在索引中嗎？
- [ ] 等值條件欄位在範圍條件之前嗎？
- [ ] 是否可以建立覆蓋索引？
- [ ] 索引的選擇性夠高嗎（> 10%）？
- [ ] 複合索引的欄位順序正確嗎？
- [ ] 是否有重複或冗餘的索引？
- [ ] 更新頻繁的欄位是否真的需要索引？
- [ ] 使用 EXPLAIN 驗證索引是否被使用？

## 參考資源

- [MySQL Indexing Cookbook](https://mysql.rjweb.org/doc.php/index_cookbook_mysql) - Rick James
- [MySQL 官方文檔：索引優化](https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html)
- [High Performance MySQL](https://www.oreilly.com/library/view/high-performance-mysql/9781492080503/) - 深入理解 MySQL 效能優化

記住：**索引不是越多越好，而是越精準越好**。每個索引都要有明確的使用場景，並且要定期檢視和優化。