+++
title = 'MySQL 分區表實戰指南：維護與最佳實踐'
date = 2025-08-25T10:00:00+08:00
draft = false
tags = ['MySQL', '資料庫', '分區表', '時間序列', '資料維護', 'SQL']
categories = ['技術筆記']
author = 'Jack'
description = '深入探討 MySQL 分區表的實際應用場景、維護策略和最佳實踐。從基礎概念到自動化管理，幫助你正確使用分區表解決實際問題。'
toc = true
weight = 1
+++

MySQL 分區表是一個常被誤解的功能。許多人認為它能顯著提升查詢性能，但實際上，分區表的主要價值在於**資料管理**而非性能優化。本文基於 Rick James 的 [Partition Maintenance](https://mysql.rjweb.org/doc.php/partitionmaint) 實戰經驗，探討分區表的正確使用方式。

## 分區表的本質與迷思

### 什麼是分區表？

分區表將一個邏輯表拆分成多個物理子表，但對應用程式而言仍是單一表。每個分區獨立存儲，可以獨立管理。

```sql
-- 一個按月分區的日誌表
CREATE TABLE logs (
    id BIGINT AUTO_INCREMENT,
    log_time DATETIME NOT NULL,
    message TEXT,
    PRIMARY KEY (id, log_time)
) PARTITION BY RANGE (TO_DAYS(log_time)) (
    PARTITION p202501 VALUES LESS THAN (TO_DAYS('2025-02-01')),
    PARTITION p202502 VALUES LESS THAN (TO_DAYS('2025-03-01')),
    PARTITION p202503 VALUES LESS THAN (TO_DAYS('2025-04-01')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

### 常見迷思破解

❌ **迷思 1：分區表能大幅提升查詢性能**
- 實際上，分區通常會增加查詢開銷
- MySQL 需要檢查多個分區，除非有明確的分區裁剪（Partition Pruning）

❌ **迷思 2：越多分區越好**
- 建議保持在 5-50 個分區之間
- 過多分區會增加元數據管理開銷

❌ **迷思 3：分區表適合所有大表**
- 只有特定場景才真正受益於分區
- 錯誤使用會導致性能下降

## 分區表的真正價值場景

### 1. 時間序列資料管理 ⭐⭐⭐⭐⭐

這是分區表最經典的應用場景：

```sql
-- 按天分區的交易記錄表
CREATE TABLE transactions (
    trans_id BIGINT,
    trans_date DATE NOT NULL,
    amount DECIMAL(10,2),
    customer_id INT,
    PRIMARY KEY (trans_date, trans_id),
    INDEX idx_customer (customer_id, trans_date)
) PARTITION BY RANGE (TO_DAYS(trans_date)) (
    PARTITION p20250820 VALUES LESS THAN (TO_DAYS('2025-08-21')),
    PARTITION p20250821 VALUES LESS THAN (TO_DAYS('2025-08-22')),
    PARTITION p20250822 VALUES LESS THAN (TO_DAYS('2025-08-23')),
    PARTITION p20250823 VALUES LESS THAN (TO_DAYS('2025-08-24')),
    PARTITION p20250824 VALUES LESS THAN (TO_DAYS('2025-08-25')),
    PARTITION p20250825 VALUES LESS THAN (TO_DAYS('2025-08-26')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- 刪除舊資料：DROP PARTITION 比 DELETE 快 1000 倍！
ALTER TABLE transactions DROP PARTITION p20250820;

-- 對比傳統 DELETE（可能需要數小時）
-- DELETE FROM transactions WHERE trans_date < '2025-08-21';
```

### 2. 二維索引優化

當需要同時對兩個範圍條件進行查詢時：

```sql
-- 地理位置查詢表
CREATE TABLE locations (
    id INT,
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    created_at DATETIME,
    PRIMARY KEY (id, latitude)
) PARTITION BY RANGE (FLOOR(latitude)) (
    PARTITION p_lat_0 VALUES LESS THAN (10),
    PARTITION p_lat_10 VALUES LESS THAN (20),
    PARTITION p_lat_20 VALUES LESS THAN (30),
    PARTITION p_lat_30 VALUES LESS THAN (40),
    PARTITION p_lat_40 VALUES LESS THAN (50),
    PARTITION p_lat_50 VALUES LESS THAN (60)
);

-- 查詢特定區域（分區裁剪會自動生效）
SELECT * FROM locations 
WHERE latitude BETWEEN 25.5 AND 35.5 
  AND longitude BETWEEN 120.5 AND 122.5;
```

### 3. 多租戶資料隔離

```sql
-- 按客戶分區的訂單表
CREATE TABLE orders (
    order_id BIGINT,
    customer_id INT NOT NULL,
    order_date DATE,
    total_amount DECIMAL(10,2),
    PRIMARY KEY (customer_id, order_id)
) PARTITION BY HASH(customer_id) 
PARTITIONS 16;  -- 16 個分區，自動分配

-- 特定客戶的查詢只會掃描一個分區
SELECT * FROM orders WHERE customer_id = 12345;
```

## 分區表設計最佳實踐

### 1. 選擇正確的分區策略

```sql
-- RANGE 分區：最常用，適合時間序列
PARTITION BY RANGE (TO_DAYS(date_column))

-- LIST 分區：適合有限的離散值
PARTITION BY LIST (region_code) (
    PARTITION p_asia VALUES IN (1, 2, 3),
    PARTITION p_europe VALUES IN (4, 5, 6),
    PARTITION p_america VALUES IN (7, 8, 9)
)

-- HASH 分區：適合均勻分配
PARTITION BY HASH(user_id) PARTITIONS 8

-- KEY 分區：類似 HASH，但使用 MySQL 內部雜湊函數
PARTITION BY KEY(column_name) PARTITIONS 10
```

### 2. 主鍵和唯一鍵設計

```sql
-- ❌ 錯誤：分區鍵不在主鍵中
CREATE TABLE wrong_example (
    id INT PRIMARY KEY,  -- 錯誤！
    created_at DATE
) PARTITION BY RANGE (TO_DAYS(created_at));

-- ✅ 正確：分區鍵必須是主鍵的一部分
CREATE TABLE correct_example (
    id INT,
    created_at DATE,
    PRIMARY KEY (id, created_at)  -- 包含分區鍵
) PARTITION BY RANGE (TO_DAYS(created_at));
```

### 3. 索引優化策略

```sql
-- 將分區鍵放在複合索引的最後
CREATE TABLE optimized_table (
    user_id INT,
    action_date DATE,
    action_type VARCHAR(50),
    PRIMARY KEY (action_date, user_id),
    INDEX idx_user_action (user_id, action_type, action_date)  -- 分區鍵在最後
) PARTITION BY RANGE (TO_DAYS(action_date));
```

## 自動化分區維護

### 1. 自動添加新分區（存儲過程）

```sql
DELIMITER $$

CREATE PROCEDURE AddDailyPartitions(
    IN table_name VARCHAR(64),
    IN days_ahead INT
)
BEGIN
    DECLARE current_date DATE DEFAULT CURDATE();
    DECLARE end_date DATE;
    DECLARE partition_date DATE;
    DECLARE partition_name VARCHAR(64);
    
    SET end_date = DATE_ADD(current_date, INTERVAL days_ahead DAY);
    SET partition_date = current_date;
    
    WHILE partition_date <= end_date DO
        SET partition_name = CONCAT('p', DATE_FORMAT(partition_date, '%Y%m%d'));
        
        SET @sql = CONCAT(
            'ALTER TABLE ', table_name,
            ' ADD PARTITION (PARTITION ', partition_name,
            ' VALUES LESS THAN (TO_DAYS(\'',
            DATE_ADD(partition_date, INTERVAL 1 DAY), '\')))'
        );
        
        -- 檢查分區是否已存在
        IF NOT EXISTS (
            SELECT 1 FROM INFORMATION_SCHEMA.PARTITIONS
            WHERE TABLE_NAME = table_name
            AND PARTITION_NAME = partition_name
        ) THEN
            PREPARE stmt FROM @sql;
            EXECUTE stmt;
            DEALLOCATE PREPARE stmt;
        END IF;
        
        SET partition_date = DATE_ADD(partition_date, INTERVAL 1 DAY);
    END WHILE;
END$$

DELIMITER ;

-- 使用方式：提前創建未來 7 天的分區
CALL AddDailyPartitions('transactions', 7);
```

### 2. 自動刪除舊分區

```sql
DELIMITER $$

CREATE PROCEDURE DropOldPartitions(
    IN table_name VARCHAR(64),
    IN days_to_keep INT
)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE partition_name VARCHAR(64);
    DECLARE partition_description VARCHAR(100);
    DECLARE cutoff_date DATE;
    
    DECLARE cur CURSOR FOR
        SELECT PARTITION_NAME, PARTITION_DESCRIPTION
        FROM INFORMATION_SCHEMA.PARTITIONS
        WHERE TABLE_NAME = table_name
        AND PARTITION_NAME NOT IN ('p_future', 'p_maxvalue')
        AND PARTITION_DESCRIPTION < TO_DAYS(DATE_SUB(CURDATE(), INTERVAL days_to_keep DAY));
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN cur;
    
    read_loop: LOOP
        FETCH cur INTO partition_name, partition_description;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        SET @sql = CONCAT('ALTER TABLE ', table_name, ' DROP PARTITION ', partition_name);
        PREPARE stmt FROM @sql;
        EXECUTE stmt;
        DEALLOCATE PREPARE stmt;
        
        SELECT CONCAT('Dropped partition: ', partition_name) AS result;
    END LOOP;
    
    CLOSE cur;
END$$

DELIMITER ;

-- 使用方式：保留最近 30 天的資料
CALL DropOldPartitions('transactions', 30);
```

### 3. 使用事件排程器自動維護

```sql
-- 啟用事件排程器
SET GLOBAL event_scheduler = ON;

-- 創建每日維護事件
CREATE EVENT IF NOT EXISTS maintain_partitions
ON SCHEDULE EVERY 1 DAY
STARTS '2025-08-26 02:00:00'
DO
BEGIN
    -- 添加未來 7 天的分區
    CALL AddDailyPartitions('transactions', 7);
    
    -- 刪除超過 90 天的舊分區
    CALL DropOldPartitions('transactions', 90);
    
    -- 記錄維護日誌
    INSERT INTO maintenance_log (task, executed_at)
    VALUES ('Partition Maintenance', NOW());
END;
```

## 分區表的陷阱與限制

### 1. 查詢性能陷阱

```sql
-- ❌ 沒有分區裁剪的查詢（掃描所有分區）
SELECT * FROM transactions 
WHERE amount > 1000;  -- 沒有使用分區鍵

-- ✅ 有效的分區裁剪
SELECT * FROM transactions 
WHERE trans_date = '2025-08-25' 
  AND amount > 1000;  -- 只掃描一個分區

-- 使用 EXPLAIN PARTITIONS 檢查分區裁剪
EXPLAIN PARTITIONS 
SELECT * FROM transactions 
WHERE trans_date BETWEEN '2025-08-20' AND '2025-08-25';
```

### 2. ALTER TABLE 限制

```sql
-- 某些 ALTER 操作會重建整個表（非常慢）
ALTER TABLE huge_partitioned_table ENGINE = InnoDB;  -- 重建所有分區！

-- 建議：對單個分區操作
ALTER TABLE huge_partitioned_table REBUILD PARTITION p20250825;
```

### 3. 外鍵約束不支援

```sql
-- ❌ 分區表不支援外鍵
CREATE TABLE orders_partitioned (
    order_id INT,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(id),  -- 錯誤！
    PRIMARY KEY (order_date, order_id)
) PARTITION BY RANGE (TO_DAYS(order_date));
```

## 實戰案例：日誌表分區管理

### 完整的日誌表設計

```sql
-- 1. 創建分區日誌表
CREATE TABLE app_logs (
    log_id BIGINT AUTO_INCREMENT,
    log_date DATE NOT NULL,
    log_time DATETIME NOT NULL,
    level ENUM('DEBUG', 'INFO', 'WARN', 'ERROR', 'FATAL'),
    message TEXT,
    app_name VARCHAR(50),
    server_ip VARCHAR(15),
    PRIMARY KEY (log_date, log_id),
    INDEX idx_level_time (level, log_time),
    INDEX idx_app (app_name, log_date)
) ENGINE=InnoDB
PARTITION BY RANGE (TO_DAYS(log_date)) (
    PARTITION p_init VALUES LESS THAN (0),
    PARTITION p_202508 VALUES LESS THAN (TO_DAYS('2025-09-01')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- 2. 創建分區管理表
CREATE TABLE partition_management (
    table_name VARCHAR(64),
    partition_type ENUM('daily', 'weekly', 'monthly'),
    retention_days INT,
    last_maintained DATETIME,
    PRIMARY KEY (table_name)
);

INSERT INTO partition_management VALUES 
('app_logs', 'daily', 90, NOW());

-- 3. 智能維護腳本
DELIMITER $$

CREATE PROCEDURE SmartPartitionMaintenance()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE v_table_name VARCHAR(64);
    DECLARE v_partition_type VARCHAR(10);
    DECLARE v_retention_days INT;
    
    DECLARE cur CURSOR FOR
        SELECT table_name, partition_type, retention_days
        FROM partition_management
        WHERE last_maintained < DATE_SUB(NOW(), INTERVAL 1 DAY);
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN cur;
    
    maintenance_loop: LOOP
        FETCH cur INTO v_table_name, v_partition_type, v_retention_days;
        IF done THEN
            LEAVE maintenance_loop;
        END IF;
        
        -- 根據類型添加分區
        CASE v_partition_type
            WHEN 'daily' THEN
                CALL AddDailyPartitions(v_table_name, 7);
            WHEN 'weekly' THEN
                CALL AddWeeklyPartitions(v_table_name, 4);
            WHEN 'monthly' THEN
                CALL AddMonthlyPartitions(v_table_name, 3);
        END CASE;
        
        -- 刪除舊分區
        CALL DropOldPartitions(v_table_name, v_retention_days);
        
        -- 更新維護時間
        UPDATE partition_management 
        SET last_maintained = NOW()
        WHERE table_name = v_table_name;
        
    END LOOP;
    
    CLOSE cur;
END$$

DELIMITER ;
```

## 監控與診斷

### 1. 分區資訊查詢

```sql
-- 查看表的所有分區
SELECT 
    PARTITION_NAME,
    PARTITION_EXPRESSION,
    PARTITION_DESCRIPTION,
    TABLE_ROWS,
    DATA_LENGTH / 1024 / 1024 AS data_size_mb,
    INDEX_LENGTH / 1024 / 1024 AS index_size_mb
FROM INFORMATION_SCHEMA.PARTITIONS
WHERE TABLE_NAME = 'transactions'
  AND TABLE_SCHEMA = DATABASE()
ORDER BY PARTITION_ORDINAL_POSITION;

-- 查看分區裁剪效果
EXPLAIN PARTITIONS 
SELECT COUNT(*) FROM transactions 
WHERE trans_date BETWEEN '2025-08-20' AND '2025-08-25';
```

### 2. 性能監控

```sql
-- 創建分區操作監控表
CREATE TABLE partition_monitor (
    id INT AUTO_INCREMENT PRIMARY KEY,
    operation VARCHAR(50),
    table_name VARCHAR(64),
    partition_name VARCHAR(64),
    rows_affected BIGINT,
    execution_time DECIMAL(10,3),
    executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 在維護程序中添加監控
-- 記錄 DROP PARTITION 的性能
SET @start_time = NOW(6);
ALTER TABLE transactions DROP PARTITION p20250820;
SET @exec_time = TIMESTAMPDIFF(MICROSECOND, @start_time, NOW(6)) / 1000000;

INSERT INTO partition_monitor (operation, table_name, partition_name, execution_time)
VALUES ('DROP', 'transactions', 'p20250820', @exec_time);
```

## 決策指南：是否使用分區表？

### ✅ 適合使用分區的場景

1. **時間序列資料**
   - 日誌、交易記錄、感測器資料
   - 需要定期刪除舊資料
   - 查詢通常包含時間範圍

2. **資料生命週期管理**
   - 有明確的資料保留策略
   - 需要快速歸檔或刪除大量資料

3. **特定的查詢模式**
   - 查詢總是包含分區鍵
   - 需要物理隔離不同類別的資料

### ❌ 不適合使用分區的場景

1. **隨機查詢模式**
   - 查詢條件多變，不固定使用分區鍵
   - 需要頻繁的跨分區 JOIN

2. **小表或中等規模表**
   - 資料量小於 1000 萬筆記錄
   - 普通索引已能滿足性能需求

3. **頻繁更新分區鍵**
   - 更新分區鍵會導致資料在分區間移動
   - 效能開銷巨大

## 總結與建議

1. **分區表不是萬能藥**：它主要解決資料管理問題，而非性能問題
2. **正確的使用場景**：時間序列資料管理是最佳應用
3. **保持簡單**：使用 RANGE 分區，控制在 50 個分區以內
4. **自動化維護**：使用存儲過程和事件排程器
5. **監控與測試**：使用 EXPLAIN PARTITIONS 驗證分區裁剪
6. **考慮替代方案**：有時候適當的索引設計比分區更有效

記住 Rick James 的金句：**"Partitioning is not a performance panacea"**（分區不是性能萬靈丹）。在決定使用分區表之前，請確保它真正解決了你的問題。

## 參考資源

- [Partition Maintenance](https://mysql.rjweb.org/doc.php/partitionmaint) - Rick James
- [MySQL 官方文檔：分區](https://dev.mysql.com/doc/refman/8.0/en/partitioning.html)
- [MySQL 分區限制](https://dev.mysql.com/doc/refman/8.0/en/partitioning-limitations.html)