+++
title = 'WoW 私服架設：MySQL/MariaDB 資料庫安裝與配置詳解'
date = 2025-08-19T11:30:00+08:00
draft = false
tags = ['WoW', 'MySQL', 'MariaDB', '資料庫', 'SQL']
categories = ['WoW私服']
author = 'Jack'
description = '完整說明如何安裝配置 MySQL/MariaDB 資料庫，建立 AzerothCore 所需的資料庫結構與初始資料'
toc = true
weight = 104
+++

## 前言

資料庫是 WoW 私服的資料中心，儲存所有角色、物品、任務、NPC 等遊戲資料。本文將詳細介紹如何設定 MySQL/MariaDB 資料庫系統。

## 資料庫架構說明

AzerothCore 使用三個主要資料庫：

1. **auth**：驗證資料庫
   - 帳號資訊
   - 登入記錄
   - IP 封鎖清單

2. **characters**：角色資料庫
   - 玩家角色資料
   - 公會資訊
   - 郵件系統
   - 拍賣場資料

3. **world**：世界資料庫
   - NPC 資料
   - 物品資訊
   - 任務內容
   - 遊戲機制

## 第一步：安裝 MariaDB

### 安裝 MariaDB Server

```bash
# 更新套件清單
apt update

# 安裝 MariaDB
apt install -y mariadb-server mariadb-client

# 檢查版本
mysql --version
```

### 啟動並設定開機自動啟動

```bash
# 啟動 MariaDB
systemctl start mariadb

# 設定開機自動啟動
systemctl enable mariadb

# 檢查服務狀態
systemctl status mariadb
```

## 第二步：安全設定

### 執行安全設定腳本

```bash
mysql_secure_installation
```

按照提示操作：
1. **Enter current password for root**: 按 Enter（預設無密碼）
2. **Set root password?**: Y，設定 root 密碼
3. **Remove anonymous users?**: Y
4. **Disallow root login remotely?**: N（允許遠端管理）
5. **Remove test database?**: Y
6. **Reload privilege tables?**: Y

### 設定 root 密碼（如果上述步驟失敗）

```bash
# 登入 MySQL
mysql -u root

# 設定 root 密碼
ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_password';
FLUSH PRIVILEGES;
EXIT;
```

## 第三步：建立 AzerothCore 使用者

### 登入 MySQL

```bash
mysql -u root -p
```

### 建立專用使用者

```sql
-- 建立 acore 使用者
CREATE USER 'acore'@'localhost' IDENTIFIED BY 'acore_password';

-- 授予所有權限
GRANT ALL PRIVILEGES ON *.* TO 'acore'@'localhost' WITH GRANT OPTION;

-- 建立遠端存取使用者（選用）
CREATE USER 'acore'@'%' IDENTIFIED BY 'acore_password';
GRANT ALL PRIVILEGES ON *.* TO 'acore'@'%' WITH GRANT OPTION;

-- 重新載入權限
FLUSH PRIVILEGES;
```

## 第四步：優化 MySQL 設定

### 編輯設定檔

```bash
vim /etc/mysql/mariadb.conf.d/50-server.cnf
```

### 加入優化參數

在 `[mysqld]` 區段加入：

```ini
[mysqld]
# 基本設定
skip-name-resolve
max_connections = 200
max_allowed_packet = 64M

# InnoDB 優化
innodb_buffer_pool_size = 2G    # 設為系統記憶體的 50%
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1

# 查詢快取
query_cache_type = 1
query_cache_size = 128M
query_cache_limit = 2M

# 緩衝區設定
key_buffer_size = 256M
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M

# 字元集
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci

# 慢查詢日誌（除錯用）
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

### 重啟 MariaDB

```bash
systemctl restart mariadb
```

## 第五步：建立資料庫

### 使用 AzerothCore 腳本建立

```bash
# 進入 AzerothCore 目錄
cd /opt/wow/azerothcore-wotlk

# 執行資料庫安裝腳本
./acore.sh db-assembler

# 導入資料庫
./acore.sh db-import
```

### 手動建立資料庫（備選方案）

```bash
# 登入 MySQL
mysql -u acore -p

# 建立資料庫
CREATE DATABASE IF NOT EXISTS `acore_auth` 
  DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

CREATE DATABASE IF NOT EXISTS `acore_characters` 
  DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

CREATE DATABASE IF NOT EXISTS `acore_world` 
  DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

# 如果使用 PlayerBots
CREATE DATABASE IF NOT EXISTS `acore_playerbots` 
  DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

EXIT;
```

## 第六步：導入初始資料

### 自動導入

```bash
cd /opt/wow/azerothcore-wotlk

# 導入所有資料庫
./acore.sh db-import
```

### 手動導入（如自動失敗）

```bash
# 進入 SQL 檔案目錄
cd /opt/wow/azerothcore-wotlk/data/sql

# 導入基礎結構
mysql -u acore -p acore_auth < base/auth_database.sql
mysql -u acore -p acore_characters < base/characters_database.sql
mysql -u acore -p acore_world < base/world_database.sql

# 導入更新
mysql -u acore -p acore_auth < updates/auth/*.sql
mysql -u acore -p acore_characters < updates/characters/*.sql
mysql -u acore -p acore_world < updates/world/*.sql
```

## 第七步：配置伺服器連線

### 編輯 authserver.conf

```bash
vim /opt/wow/server/etc/authserver.conf
```

修改資料庫連線設定：

```conf
# MySQL 連線設定
LoginDatabaseInfo = "127.0.0.1;3306;acore;acore_password;acore_auth"

# 更新設定
Updates.EnableDatabases = 1
Updates.AutoSetup = 1
Updates.Redundancy = 1
```

### 編輯 worldserver.conf

```bash
vim /opt/wow/server/etc/worldserver.conf
```

修改資料庫連線設定：

```conf
# MySQL 連線設定
LoginDatabaseInfo     = "127.0.0.1;3306;acore;acore_password;acore_auth"
WorldDatabaseInfo     = "127.0.0.1;3306;acore;acore_password;acore_world"
CharacterDatabaseInfo = "127.0.0.1;3306;acore;acore_password;acore_characters"

# 資料庫連線池
Database.WorkerThreads = 1
Database.LoginDatabase.WorkerThreads     = 1
Database.WorldDatabase.WorkerThreads     = 1
Database.CharacterDatabase.WorkerThreads = 1
```

## 第八步：安裝 HeidiSQL（Windows 端）

### 下載與安裝

1. 下載 [HeidiSQL](https://www.heidisql.com/download.php)
2. 執行安裝程式
3. 使用預設設定完成安裝

### 設定連線

1. 開啟 HeidiSQL
2. 建立新連線：
   - **網路類型**：MariaDB or MySQL (TCP/IP)
   - **主機名稱**：虛擬機 IP
   - **使用者**：acore
   - **密碼**：acore_password
   - **埠口**：3306

### 允許遠端連線（如需要）

```bash
# 編輯 MariaDB 設定
vim /etc/mysql/mariadb.conf.d/50-server.cnf

# 註解掉 bind-address
# bind-address = 127.0.0.1

# 重啟服務
systemctl restart mariadb
```

## 第九步：資料庫維護

### 備份資料庫

建立備份腳本：

```bash
vim /opt/wow/backup_db.sh
```

加入內容：

```bash
#!/bin/bash

BACKUP_DIR="/opt/wow/backups"
DATE=$(date +%Y%m%d_%H%M%S)
MYSQL_USER="acore"
MYSQL_PASS="acore_password"

# 建立備份目錄
mkdir -p $BACKUP_DIR

# 備份各資料庫
mysqldump -u$MYSQL_USER -p$MYSQL_PASS acore_auth > $BACKUP_DIR/auth_$DATE.sql
mysqldump -u$MYSQL_USER -p$MYSQL_PASS acore_characters > $BACKUP_DIR/characters_$DATE.sql
mysqldump -u$MYSQL_USER -p$MYSQL_PASS acore_world > $BACKUP_DIR/world_$DATE.sql

# 壓縮備份
tar -czf $BACKUP_DIR/backup_$DATE.tar.gz $BACKUP_DIR/*_$DATE.sql
rm $BACKUP_DIR/*_$DATE.sql

# 保留最近 7 天的備份
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: backup_$DATE.tar.gz"
```

設定執行權限：
```bash
chmod +x /opt/wow/backup_db.sh
```

### 還原資料庫

```bash
# 解壓縮備份
tar -xzf backup_20240101_120000.tar.gz

# 還原資料庫
mysql -u acore -p acore_auth < auth_20240101_120000.sql
mysql -u acore -p acore_characters < characters_20240101_120000.sql
mysql -u acore -p acore_world < world_20240101_120000.sql
```

### 設定自動備份

```bash
# 編輯 crontab
crontab -e

# 每天凌晨 3 點自動備份
0 3 * * * /opt/wow/backup_db.sh
```

## 常見問題排除

### 問題 1：無法連線到資料庫

```bash
# 檢查服務狀態
systemctl status mariadb

# 檢查錯誤日誌
tail -f /var/log/mysql/error.log

# 測試連線
mysql -u acore -p -h localhost
```

### 問題 2：權限錯誤

```sql
-- 重新授權
GRANT ALL PRIVILEGES ON *.* TO 'acore'@'localhost' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'acore'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### 問題 3：字元集問題

```sql
-- 修改資料庫字元集
ALTER DATABASE acore_world 
  DEFAULT CHARACTER SET utf8mb4 
  COLLATE utf8mb4_general_ci;
```

## 效能監控

### 查看連線狀態

```sql
-- 查看當前連線
SHOW PROCESSLIST;

-- 查看連線統計
SHOW STATUS LIKE 'Threads%';
```

### 查看慢查詢

```bash
# 查看慢查詢日誌
tail -f /var/log/mysql/slow.log
```

### 優化表格

```sql
-- 優化所有表格
OPTIMIZE TABLE acore_characters.*;
OPTIMIZE TABLE acore_auth.*;
```

## 下一步

資料庫設定完成！我們現在擁有：

✅ 運行中的 MariaDB 伺服器  
✅ 建立的三個核心資料庫  
✅ 正確的使用者權限設定  
✅ 資料庫備份機制  

下一篇文章將介紹 PlayerBots 模組的詳細設定，讓你的私服充滿 AI 玩家。

## 實用 SQL 指令

```sql
-- 查看資料庫大小
SELECT 
  table_schema AS 'Database',
  ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
GROUP BY table_schema;

-- 查看資料表大小
SELECT 
  table_name AS 'Table',
  ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Size (MB)'
FROM information_schema.tables
WHERE table_schema = 'acore_world'
ORDER BY (data_length + index_length) DESC;
```

準備好設定 PlayerBots 了嗎？讓我們繼續！