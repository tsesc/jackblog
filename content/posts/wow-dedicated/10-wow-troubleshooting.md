+++
title = 'WoW 私服架設：常見問題與疑難排解完整指南'
date = 2025-08-19T14:30:00+08:00
draft = false
tags = ['WoW', '疑難排解', '錯誤修復', '維護', '優化']
categories = ['WoW私服']
author = 'Jack'
description = '整理 WoW 私服架設與運營過程中的常見問題、錯誤訊息解決方案，以及伺服器維護技巧'
toc = true
weight = 10
+++

## 前言

架設私服過程中難免遇到各種問題。本文整理了最常見的錯誤與解決方案，幫助你快速排除故障。

## 伺服器啟動問題

### Authserver 無法啟動

**錯誤訊息**：
```
Cannot connect to database
```

**解決方案**：
```bash
# 檢查 MySQL 服務
systemctl status mariadb

# 確認資料庫存在
mysql -u acore -p -e "SHOW DATABASES;"

# 檢查連線設定
grep LoginDatabaseInfo /opt/wow/server/etc/authserver.conf
```

### Worldserver 崩潰

**錯誤訊息**：
```
Segmentation fault (core dumped)
```

**解決方案**：
```bash
# 檢查資料檔案完整性
ls -la /opt/wow/server/data/

# 檢查記憶體
free -h

# 增加 swap
fallocate -l 4G /swapfile
mkswap /swapfile
swapon /swapfile
```

### 找不到地圖檔案

**錯誤訊息**：
```
Map file './maps/000.map' does not exist
```

**解決方案**：
```bash
# 確認資料路徑
vim /opt/wow/server/etc/worldserver.conf
# DataDir = "/opt/wow/server/data"

# 重新下載資料檔
cd /opt/wow/server/data
wget [資料檔網址]
unzip data.zip
```

## 資料庫問題

### 資料庫連線失敗

**錯誤訊息**：
```
Access denied for user 'acore'@'localhost'
```

**解決方案**：
```sql
-- 重設密碼
ALTER USER 'acore'@'localhost' IDENTIFIED BY 'new_password';

-- 重新授權
GRANT ALL PRIVILEGES ON *.* TO 'acore'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### 資料庫更新失敗

**錯誤訊息**：
```
Applying update "xxx.sql" FAILED
```

**解決方案**：
```bash
# 手動執行更新
mysql -u acore -p acore_world < update.sql

# 清理更新狀態
mysql -u acore -p acore_world -e "DELETE FROM updates WHERE name='xxx.sql';"
```

### 資料庫損壞

**症狀**：角色無法載入、物品消失

**解決方案**：
```bash
# 修復資料表
mysqlcheck -u acore -p --auto-repair --all-databases

# 從備份還原
mysql -u acore -p acore_characters < backup.sql
```

## 客戶端連線問題

### 無法連線到伺服器

**檢查清單**：

1. **Realmlist 設定**
```
檢查 Data/enUS/realmlist.wtf
確認 IP 正確
```

2. **防火牆規則**
```bash
# Linux 防火牆
iptables -I INPUT -p tcp --dport 3724 -j ACCEPT
iptables -I INPUT -p tcp --dport 8085 -j ACCEPT
```

3. **網路連通性**
```cmd
ping 伺服器IP
telnet 伺服器IP 3724
```

### 卡在 "Authenticating"

**可能原因**：
- authserver 未運行
- 資料庫連線問題
- 版本不符

**解決方案**：
```bash
# 檢查 authserver
ps aux | grep authserver

# 查看日誌
tail -f /opt/wow/server/logs/Auth.log
```

### 世界伺服器已滿

**錯誤訊息**：
```
World server is full
```

**解決方案**：
```conf
# 編輯 worldserver.conf
PlayerLimit = 100  # 增加上限
```

## PlayerBots 問題

### 機器人不出現

```bash
# 檢查模組載入
grep PlayerBot /opt/wow/server/logs/worldserver.log

# 確認機器人帳號
mysql -u acore -p acore_auth -e "SELECT * FROM account WHERE username LIKE 'rndbot%';"
```

### 機器人行為異常

```conf
# 調整 AI 設定
AiPlayerbot.BotFollowDistance = 100
AiPlayerbot.ReactDelay = 100
AiPlayerbot.IterationsPerTick = 10
```

## 效能優化

### CPU 使用率過高

```bash
# 限制 CPU 使用
nice -n 10 ./worldserver

# 調整更新頻率
# worldserver.conf
MapUpdate.Threads = 2
MapUpdateInterval = 200
```

### 記憶體洩漏

```bash
# 監控記憶體
watch -n 1 'ps aux | grep worldserver'

# 定期重啟
crontab -e
0 4 * * * /opt/wow/restart_server.sh
```

### 延遲過高

```conf
# 優化網路設定
Compression = 1
CompressionLevel = 6

# 減少視距
Visibility.Distance.Continents = 60
```

## 日誌分析

### 查看錯誤日誌

```bash
# 即時監控
tail -f /opt/wow/server/logs/worldserver.log

# 搜尋錯誤
grep ERROR /opt/wow/server/logs/*.log

# 分析崩潰
gdb /opt/wow/server/bin/worldserver core
```

### 常見錯誤代碼

| 錯誤碼 | 含義 | 解決方法 |
|--------|------|----------|
| 0x001  | 記憶體不足 | 增加 RAM 或 Swap |
| 0x002  | 資料庫錯誤 | 檢查 MySQL 連線 |
| 0x003  | 檔案缺失 | 重新下載資料檔 |
| 0x004  | 版本錯誤 | 使用正確版本客戶端 |

## 維護腳本

### 自動重啟腳本

```bash
#!/bin/bash
# auto_restart.sh

check_server() {
    if ! pgrep -x "worldserver" > /dev/null; then
        echo "Server down, restarting..."
        cd /opt/wow/server/bin
        screen -dmS wow-world ./worldserver
    fi
}

while true; do
    check_server
    sleep 60
done
```

### 定期備份腳本

```bash
#!/bin/bash
# daily_backup.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/wow"

# 備份資料庫
mysqldump -u acore -p acore_auth > $BACKUP_DIR/auth_$DATE.sql
mysqldump -u acore -p acore_characters > $BACKUP_DIR/char_$DATE.sql
mysqldump -u acore -p acore_world > $BACKUP_DIR/world_$DATE.sql

# 保留 7 天
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete
```

## 緊急修復指令

### 重置角色位置

```sql
-- 將角色傳送到主城
UPDATE characters SET 
    position_x = -8913.23,
    position_y = 554.633,
    position_z = 93.7944,
    map = 0
WHERE name = '角色名';
```

### 修復損壞的任務

```sql
-- 清除角色任務
DELETE FROM character_queststatus WHERE guid = 角色GUID;
```

### 重置副本

```
.instance unbind all player
.instance reset all
```

## 角色備份與還原

### 使用 GM 指令備份角色

你可以使用 `.pdump` 指令來備份和還原單個角色：

#### 備份角色

在遊戲中：
```
.pdump write [檔名] [角色名稱或GUID]
.pdump write MyCharBackup Painbow
```

在伺服器控制台：
```
pdump write MyCharBackup Painbow
```

備份檔案會儲存在：`~/azerothcore-wotlk/env/dist/bin/`

#### 還原角色

在遊戲中：
```
.pdump load [檔名] [帳號名稱] [新角色名] [新GUID]
.pdump load MyCharBackup myaccount
```

在伺服器控制台：
```
pdump load MyCharBackup myaccount
```

**注意事項**：
- 檔名可以任意命名，但還原時會使用原始角色名稱
- 如果角色已存在，可能會提示重新命名
- 建議在還原前先刪除同名角色

### 自動化備份腳本

建立每日自動備份重要角色的腳本：

```bash
#!/bin/bash
# backup_characters.sh

BACKUP_DIR="/opt/wow/character_backups"
DATE=$(date +%Y%m%d_%H%M%S)
CHARACTERS=("Painbow" "MyHealer" "MyTank")

mkdir -p $BACKUP_DIR

for char in "${CHARACTERS[@]}"; do
    echo "Backing up $char..."
    cd /opt/wow/azerothcore-wotlk/env/dist/bin
    ./worldserver --backup "$char" "$BACKUP_DIR/${char}_$DATE.pbd"
done

# 保留最近 30 天的備份
find $BACKUP_DIR -name "*.pbd" -mtime +30 -delete

echo "Character backup completed!"
```

### 批量角色管理

#### 匯出所有角色列表

```sql
-- 查看所有角色
SELECT 
    c.guid,
    c.name,
    c.level,
    c.race,
    c.class,
    a.username as account
FROM characters c
JOIN account a ON c.account = a.id
ORDER BY c.level DESC;
```

#### 轉移角色到其他帳號

```sql
-- 轉移角色到其他帳號
UPDATE characters 
SET account = (SELECT id FROM account WHERE username = 'newaccount')
WHERE name = 'CharacterName';
```

## 預防措施

1. **定期備份**：每日自動備份資料庫和重要角色
2. **監控系統**：使用監控工具追蹤效能
3. **測試環境**：先在測試服測試更新
4. **文檔記錄**：記錄所有修改和設定
5. **版本控制**：使用 Git 管理設定檔

## 結語

恭喜你完成了整個 WoW 私服架設系列！現在你已經掌握了：

✅ 完整的伺服器架設流程  
✅ 各種模組的安裝方法  
✅ 常見問題的解決方案  
✅ 伺服器維護技巧  

記住，運營私服需要持續的學習和優化。加入 AzerothCore 社群，與其他管理員交流經驗，讓你的私服越來越好！

祝你的艾澤拉斯之旅愉快！