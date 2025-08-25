+++
title = 'MySQL 黃金法則完全指南：資深 DBA 的經驗結晶'
date = 2025-08-25T11:00:00+08:00
draft = false
tags = ['MySQL', '資料庫', '效能優化', 'DBA', '最佳實踐', 'SQL']
categories = ['技術筆記']
author = 'Jack'
description = '彙整 MySQL 專家 Rick James 數十年經驗的黃金法則。從效能調校到架構設計，從索引優化到硬體配置，這是每個 MySQL 開發者都應該知道的實戰指南。'
toc = true
weight = 1
+++

這篇文章整理了 MySQL 專家 Rick James 的 [Rules of Thumb (RoTs)](https://mysql.rjweb.org/doc.php/ricksrots) - 數十年實戰經驗濃縮成的黃金法則。這些法則不是死板的規定，而是經過無數生產環境驗證的最佳實踐。

> "**Count the disk hits!**" - 這是優化 MySQL 最重要的原則

## 一、記憶體配置黃金比例 💾

### InnoDB Buffer Pool 法則

```ini
# my.cnf 配置
innodb_buffer_pool_size = [RAM的70%]  # 最重要的設定！

# 範例：32GB RAM 的伺服器
innodb_buffer_pool_size = 22G
```

**為什麼是 70%？**
- 留 30% 給作業系統和其他程序
- 避免 swap（絕對不要讓 MySQL 使用 swap）
- 保留空間給連線緩衝區和臨時表

### 其他記憶體設定

```ini
# 臨時表（RAM 的 1%）
tmp_table_size = 320M
max_heap_table_size = 320M  # 必須與 tmp_table_size 相同

# 連線緩衝區（每個連線）
sort_buffer_size = 2M        # 不要超過 2M
read_buffer_size = 2M        # 順序掃描緩衝
join_buffer_size = 2M        # JOIN 操作緩衝

# 執行緒快取
thread_cache_size = 10       # 小而非零的值

# 關閉查詢快取（MySQL 8.0 已移除）
query_cache_type = 0
query_cache_size = 0
```

### 記憶體使用監控

```sql
-- 檢查 Buffer Pool 使用率
SHOW STATUS LIKE 'Innodb_buffer_pool%';

-- 理想情況：
-- Innodb_buffer_pool_pages_free 不應該接近 0
-- Innodb_buffer_pool_wait_free = 0（不應該等待空閒頁）
```

## 二、索引設計的十大法則 🔍

### 法則 1：複合索引的黃金順序

```sql
-- WHERE 子句分析
WHERE status = 'active'        -- 等值條件
  AND type = 'premium'         -- 等值條件
  AND created > '2025-01-01'   -- 範圍條件
  ORDER BY priority;           -- 排序

-- 正確的索引順序
INDEX idx_optimal (status, type, created, priority)
-- 等值 → 範圍 → 排序
```

### 法則 2：索引選擇性原則

```sql
-- 計算選擇性
SELECT 
    COUNT(DISTINCT column) / COUNT(*) AS selectivity
FROM table_name;

-- 選擇性指標：
-- > 0.9  極佳（適合建立索引）
-- 0.5-0.9 良好
-- 0.1-0.5 一般（考慮複合索引）
-- < 0.1  差（避免單獨索引）
```

### 法則 3：覆蓋索引優先

```sql
-- 查詢
SELECT user_id, username, email 
FROM users 
WHERE status = 'active';

-- 覆蓋索引（包含所有需要的欄位）
INDEX idx_covering (status, user_id, username, email)
-- 完全避免回表查詢！
```

### 法則 4：避免索引陷阱

```sql
-- ❌ 錯誤：函數操作導致索引失效
WHERE YEAR(created_date) = 2025
WHERE DATE_FORMAT(created_date, '%Y-%m') = '2025-08'

-- ✅ 正確：保持欄位原始形態
WHERE created_date >= '2025-01-01' 
  AND created_date < '2026-01-01'
WHERE created_date >= '2025-08-01' 
  AND created_date < '2025-09-01'

-- ❌ 錯誤：隱式類型轉換
WHERE phone = 123456  -- phone 是 VARCHAR

-- ✅ 正確：類型一致
WHERE phone = '123456'
```

### 法則 5：五個欄位上限

```sql
-- 複合索引不要超過 5 個欄位
INDEX idx_too_many (a, b, c, d, e)     -- 極限
INDEX idx_way_too_many (a, b, c, d, e, f)  -- 太多了！

-- 原因：
-- 1. 索引維護成本增加
-- 2. 記憶體使用增加
-- 3. 優化器可能誤判
```

## 三、查詢優化的實戰法則 ⚡

### 法則 6：20% 規則

```sql
-- 當需要超過 20% 的資料時，全表掃描比索引快
SELECT * FROM large_table WHERE status != 'deleted';
-- 如果 deleted 只佔 5%，那需要 95% 的資料
-- MySQL 會選擇全表掃描（正確的選擇）
```

### 法則 7：批次操作最佳大小

```sql
-- 單筆插入（慢）
INSERT INTO table VALUES (1, 'a');
INSERT INTO table VALUES (2, 'b');

-- 批次插入（快，但不要太大）
INSERT INTO table VALUES 
    (1, 'a'), (2, 'b'), ... (1000, 'zzz');  -- 100-1000 筆最佳

-- 超過 1000 筆應該分批
-- 原因：避免鎖定時間過長，影響並發
```

### 法則 8：JOIN vs 子查詢

```sql
-- ❌ 子查詢（通常較慢）
SELECT * FROM orders 
WHERE customer_id IN (
    SELECT id FROM customers WHERE country = 'TW'
);

-- ✅ JOIN（通常較快）
SELECT o.* FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
WHERE c.country = 'TW';

-- 例外：EXISTS 有時比 JOIN 好
SELECT * FROM orders o
WHERE EXISTS (
    SELECT 1 FROM customers c 
    WHERE c.id = o.customer_id AND c.country = 'TW'
);
```

### 法則 9：LIMIT 優化

```sql
-- ❌ 深度分頁問題
SELECT * FROM posts ORDER BY id LIMIT 1000000, 20;
-- 需要掃描 1000020 筆！

-- ✅ 使用延遲關聯
SELECT p.* FROM posts p
INNER JOIN (
    SELECT id FROM posts ORDER BY id LIMIT 1000000, 20
) AS tmp USING(id);

-- ✅✅ 最佳：記住位置
SELECT * FROM posts 
WHERE id > last_seen_id 
ORDER BY id 
LIMIT 20;
```

## 四、資料類型選擇法則 📊

### 法則 10：最小化原則

```sql
-- 整數類型選擇
TINYINT    -- -128 到 127（或 0-255 UNSIGNED）
SMALLINT   -- ±32K
MEDIUMINT  -- ±8M
INT        -- ±2B
BIGINT     -- ±9×10^18

-- 實例：年齡
age TINYINT UNSIGNED  -- 0-255 夠用

-- 實例：訂單數量
quantity SMALLINT UNSIGNED  -- 0-65535 通常夠用

-- 實例：用戶 ID
user_id INT UNSIGNED  -- 0-42億，足夠大部分應用
```

### 法則 11：字串類型策略

```sql
-- VARCHAR vs CHAR
CHAR(10)     -- 固定長度，適合：國家代碼、郵遞區號
VARCHAR(255) -- 可變長度，適合：姓名、地址

-- 長度設定原則
email VARCHAR(100)    -- Email 很少超過 100
phone VARCHAR(20)     -- 國際電話格式
username VARCHAR(30)  -- 使用者名稱
password_hash CHAR(60) -- bcrypt 固定 60 字元

-- TEXT 類型謹慎使用
TINYTEXT   -- 255 bytes
TEXT       -- 64KB
MEDIUMTEXT -- 16MB
LONGTEXT   -- 4GB（避免使用）
```

### 法則 12：時間類型選擇

```sql
-- 日期時間存儲
DATE          -- 只需要日期
DATETIME      -- 需要日期和時間
TIMESTAMP     -- 需要時區轉換（自動 UTC）

-- 最佳實踐
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

-- 避免存儲計算值
age INT  -- ❌ 會過期
birthdate DATE  -- ✅ 永遠正確
```

### 法則 13：金額存儲

```sql
-- ❌ 錯誤：浮點數不精確
price FLOAT
price DOUBLE

-- ✅ 正確：定點數
price DECIMAL(10,2)    -- 一般商品
price DECIMAL(13,4)    -- 金融交易（Rick 推薦）

-- 或者用整數（分）
price_cents INT        -- 存儲分，顯示時除以 100
```

## 五、硬體與系統配置法則 🖥️

### 法則 14：磁碟 I/O 預估

```
傳統硬碟 (HDD)：~100 IOPS
SSD：~1000 IOPS
NVMe SSD：~10000+ IOPS

-- 查詢需要的 IOPS 計算
1. 統計查詢的磁碟讀取次數
2. 乘以 QPS（每秒查詢數）
3. 加上寫入的 IOPS（通常是讀取的 20-30%）
```

### 法則 15：CPU 核心利用

```sql
-- 單一連線只能使用一個 CPU 核心
-- 但是：
-- 4 核心 = 可以同時處理 4 個查詢
-- 8 核心 = 可以同時處理 8 個查詢

-- 檢查並發查詢數
SHOW STATUS LIKE 'Threads_running';
-- 如果 > 10，可能有嚴重問題
-- 如果 > CPU 核心數 × 2，必定有問題
```

### 法則 16：連線數設定

```ini
# 連線數設定
max_connections = 200  # 預設 151，通常夠用

# 計算方式：
# max_connections = (可用RAM - 全域緩衝) / 每連線記憶體
# 每連線約需 1-3MB

# 監控連線使用
SHOW STATUS LIKE 'Max_used_connections';
# 應該 < max_connections × 0.8
```

## 六、架構設計法則 🏗️

### 法則 17：正規化 vs 反正規化

```sql
-- 適度正規化（通常到第三正規化）
-- 但不要過度正規化

-- ❌ 過度正規化範例
-- users, user_emails, user_phones, user_addresses...
-- 每個屬性一個表，JOIN 地獄

-- ✅ 實用設計
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100),
    phone VARCHAR(20),
    -- 常用欄位放一起
    -- 不常用的才分表
);

-- 有時反正規化是對的
CREATE TABLE order_summary (
    order_id INT PRIMARY KEY,
    total_amount DECIMAL(10,2),  -- 冗餘但快
    item_count INT,               -- 冗餘但實用
    -- 避免每次都要 JOIN 和 SUM
);
```

### 法則 18：表的數量限制

```
合理範圍：< 100 個表
警戒線：1000 個表（設計可能有問題）
危險區：10000+ 個表（系統會變慢）

-- 太多表的徵兆：
-- 1. 每個客戶一個表（錯誤！）
-- 2. 每天一個表（考慮分區）
-- 3. 過度分割（user_2025_08_25）
```

### 法則 19：主鍵設計原則

```sql
-- ✅ 好的主鍵
id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY

-- ✅ 複合主鍵（多對多關聯）
PRIMARY KEY (user_id, role_id)

-- ❌ 糟糕的主鍵
email VARCHAR(100) PRIMARY KEY  -- 可能變更
uuid CHAR(36) PRIMARY KEY       -- 太大，隨機插入

-- UUID 的正確用法
id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
uuid BINARY(16) UNIQUE,  -- 轉為二進位存儲
INDEX (uuid)
```

## 七、事務與鎖定法則 🔒

### 法則 20：事務大小原則

```sql
-- 事務持續時間：< 5 秒
-- 事務大小：100-1000 筆操作

START TRANSACTION;
-- 100-1000 筆操作
COMMIT;

-- ❌ 錯誤：超大事務
START TRANSACTION;
DELETE FROM logs WHERE created < '2024-01-01';  -- 刪除一年資料
COMMIT;

-- ✅ 正確：分批處理
REPEAT
    DELETE FROM logs 
    WHERE created < '2024-01-01' 
    LIMIT 1000;
UNTIL ROW_COUNT() = 0 END REPEAT;
```

### 法則 21：鎖定優化

```sql
-- 使用 SELECT ... FOR UPDATE 要小心
BEGIN;
SELECT * FROM inventory 
WHERE product_id = 123 
FOR UPDATE;  -- 鎖定這一行
-- 快速完成操作
UPDATE inventory SET quantity = quantity - 1 
WHERE product_id = 123;
COMMIT;

-- 避免間隙鎖
-- 確保查詢使用唯一索引或主鍵
```

## 八、監控與維護法則 📈

### 法則 22：慢查詢設定

```ini
# 慢查詢日誌（必開！）
slow_query_log = ON
long_query_time = 2  # 2 秒
log_queries_not_using_indexes = ON

# 分析慢查詢
# pt-query-digest slow.log
```

### 法則 23：關鍵指標監控

```sql
-- 1. Buffer Pool 命中率（應該 > 99%）
SELECT 
    (1 - (Innodb_buffer_pool_reads / Innodb_buffer_pool_read_requests)) * 100 
    AS buffer_pool_hit_rate
FROM (
    SELECT 
        VARIABLE_VALUE AS Innodb_buffer_pool_reads
    FROM INFORMATION_SCHEMA.GLOBAL_STATUS 
    WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads'
) AS reads,
(
    SELECT 
        VARIABLE_VALUE AS Innodb_buffer_pool_read_requests
    FROM INFORMATION_SCHEMA.GLOBAL_STATUS 
    WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests'
) AS requests;

-- 2. 執行緒監控
SHOW STATUS LIKE 'Threads%';
-- Threads_running < 10（正常）
-- Threads_running > 30（問題嚴重）

-- 3. 臨時表監控
SHOW STATUS LIKE 'Created_tmp%';
-- Created_tmp_disk_tables 應該很少
```

### 法則 24：定期維護任務

```sql
-- 1. 更新統計資訊（每週）
ANALYZE TABLE table_name;

-- 2. 優化表（每月，如果有大量刪除）
OPTIMIZE TABLE table_name;  -- 會鎖表，小心使用

-- 3. 檢查表完整性（每季）
CHECK TABLE table_name;
```

## 九、安全性法則 🔐

### 法則 25：最小權限原則

```sql
-- 應用程式帳號（最小權限）
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.* TO 'app_user'@'localhost';

-- 唯讀帳號（報表用）
CREATE USER 'readonly'@'%' IDENTIFIED BY 'strong_password';
GRANT SELECT ON mydb.* TO 'readonly'@'%';

-- 備份帳號
CREATE USER 'backup'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, LOCK TABLES, SHOW VIEW ON *.* TO 'backup'@'localhost';
```

### 法則 26：敏感資料處理

```sql
-- ❌ 絕不存儲
-- 信用卡號（使用支付閘道）
-- 明文密碼（使用 bcrypt/argon2）

-- ✅ 加密存儲
-- 使用應用層加密
-- 或 MySQL 的透明加密（TDE）

-- 資料脫敏
SELECT 
    CONCAT(LEFT(phone, 3), '****', RIGHT(phone, 4)) AS masked_phone,
    CONCAT(LEFT(email, 2), '***@***', RIGHT(email, 4)) AS masked_email
FROM users;
```

## 十、反面教材：絕對要避免的事 ⚠️

### 不要做的事情清單

```sql
-- ❌ 1. SELECT * （浪費資源）
SELECT * FROM large_table;

-- ❌ 2. 沒有 WHERE 的 UPDATE/DELETE（災難）
UPDATE users SET status = 'active';  -- 更新所有！
DELETE FROM logs;  -- 刪除所有！

-- ❌ 3. 使用 OFFSET 做深度分頁
SELECT * FROM posts LIMIT 1000000, 20;

-- ❌ 4. 在迴圈中查詢（N+1 問題）
for user_id in user_ids:
    SELECT * FROM orders WHERE user_id = ?

-- ❌ 5. 儲存計算值
age INT,  -- 會過時
total_orders INT,  -- 會不同步

-- ❌ 6. 使用 OR 連接不同欄位
WHERE phone = '123' OR email = 'test@example.com'
-- 無法有效使用索引

-- ❌ 7. 模糊查詢開頭
WHERE name LIKE '%jack'

-- ❌ 8. 強制索引提示
SELECT * FROM users FORCE INDEX (idx_email)
-- 讓優化器自己決定
```

## 十一、效能問題診斷速查表 🔧

### 常見問題與解決方案

| 症狀 | 可能原因 | 解決方案 |
|------|---------|----------|
| 查詢突然變慢 | 資料量增長 | 檢查索引、考慮分區 |
| CPU 100% | 缺少索引、錯誤查詢 | 檢查慢查詢日誌 |
| 記憶體不足 | Buffer Pool 太大 | 調整為 RAM 的 70% |
| 大量鎖等待 | 長事務 | 縮短事務、優化查詢 |
| 磁碟 I/O 高 | Buffer Pool 太小 | 增加記憶體或優化查詢 |
| 連線數爆滿 | 連線洩漏 | 檢查應用程式連線池 |

### 快速診斷命令

```sql
-- 1. 查看當前查詢
SHOW PROCESSLIST;

-- 2. 查看鎖等待
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;

-- 3. 查看表狀態
SHOW TABLE STATUS LIKE 'table_name';

-- 4. 查看索引使用情況
SELECT * FROM sys.schema_unused_indexes;

-- 5. 查看熱點表
SELECT * FROM sys.schema_table_statistics_with_buffer;
```

## 十二、版本遷移注意事項 📝

### MySQL 5.7 → 8.0 重要變更

```sql
-- 1. 查詢快取已移除
-- 2. 預設字元集改為 utf8mb4
-- 3. 預設認證插件改變
-- 4. GROUP BY 不再隱式排序
-- 5. JSON 功能大幅增強

-- 升級前檢查
SELECT VERSION();
mysqlcheck -u root -p --all-databases --check-upgrade
```

## 經驗總結：Rick 的智慧箴言 💡

1. **"Count the disk hits!"** - 永遠關注磁碟 I/O
2. **"INDEXes are good; COMPOSITE INDEXes are great!"** - 複合索引更強大
3. **"Don't queue it, just do it"** - 資料庫不是消息隊列
4. **"Normalize, but don't over-normalize"** - 適度正規化
5. **"70% of RAM for Buffer Pool"** - 記憶體配置黃金比例
6. **"Avoid EAV schemas"** - 避免實體-屬性-值模式
7. **"Use the smallest practical datatype"** - 資料類型最小化
8. **"Transactions should be small and fast"** - 事務要短小精悍
9. **"Monitor, but don't over-monitor"** - 監控要適度
10. **"When in doubt, EXPLAIN"** - 有疑問就用 EXPLAIN

## 持續學習資源 📚

- [Rick James 的 MySQL 文件](https://mysql.rjweb.org/) - 實戰經驗寶庫
- [MySQL 官方文檔](https://dev.mysql.com/doc/) - 權威參考
- [Percona 部落格](https://www.percona.com/blog/) - 深度技術文章
- [High Performance MySQL](https://www.oreilly.com/library/view/high-performance-mysql/9781492080503/) - 經典書籍

記住：**這些法則是指南，不是教條**。每個系統都有其特殊性，要根據實際情況調整。但當你不確定時，這些法則能為你指明方向。

> "In MySQL, as in life, there are no absolute truths, only Rules of Thumb." - Rick James