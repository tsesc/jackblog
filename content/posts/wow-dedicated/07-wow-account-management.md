+++
title = 'WoW 私服架設：遊戲帳號與權限管理完整指南'
date = 2025-08-19T13:00:00+08:00
draft = false
tags = ['WoW', '帳號管理', 'GM權限', '指令', '資料庫']
categories = ['WoW私服']
author = 'Jack'
description = '詳細說明如何管理遊戲帳號、設定 GM 權限、使用管理指令，以及帳號安全維護'
toc = true
weight = 7
+++

## 前言

正確的帳號管理是私服運營的基礎。本文將介紹如何建立帳號、設定權限、使用 GM 指令等重要功能。

## 建立遊戲帳號

### 透過控制台建立

連線到 worldserver 控制台：

```bash
screen -r wow-world
```

建立帳號指令：

```
account create [使用者名稱] [密碼]
account create admin admin123
account create player1 pass123
```

### 透過資料庫建立

```sql
-- 登入 MySQL
mysql -u acore -p acore_auth

-- 建立帳號
INSERT INTO account (username, sha_pass_hash, email, expansion) 
VALUES ('testuser', SHA1(CONCAT(UPPER('testuser'), ':', UPPER('password'))), 
        'test@example.com', 2);
```

## GM 權限等級

### 權限等級說明

- **0** - 一般玩家
- **1** - 初級 GM（基本指令）
- **2** - 中級 GM（管理指令）
- **3** - 高級 GM（所有指令）

### 設定 GM 權限

```
account set gmlevel [使用者名稱] [等級] [領域ID]
account set gmlevel admin 3 -1  # -1 表示所有領域
account set gmlevel helper 1 -1
```

## 常用 GM 指令

### 傳送指令
```
.tele [地點名稱]        # 傳送到指定地點
.summon [玩家名稱]      # 召喚玩家
.appear [玩家名稱]      # 傳送到玩家身邊
.gps                    # 顯示當前座標
```

### 物品指令
```
.additem [物品ID] [數量]  # 添加物品
.additemset [套裝ID]      # 添加套裝
.send items [玩家] [標題] [內容] [物品ID:數量]  # 郵寄物品
```

### 等級與技能
```
.levelup [等級數]        # 提升等級
.learn [技能ID]          # 學習技能
.learn all               # 學習所有技能
.maxskill                # 技能熟練度最大化
```

### 金錢與聲望
```
.modify money [數量]     # 修改金錢（單位：銅幣）
.modify rep [陣營ID] [數量]  # 修改聲望
```

## 帳號安全管理

### 密碼政策

```conf
# worldserver.conf 設定
Account.PasswordComplexity = 2  # 0=無限制, 1=簡單, 2=複雜
Account.PasswordLength.Min = 8
Account.PasswordLength.Max = 20
```

### IP 限制

```sql
-- 設定 IP 白名單
INSERT INTO ip_banned (ip, bandate, unbandate, bannedby, banreason) 
VALUES ('192.168.1.100', UNIX_TIMESTAMP(), 0, 'Admin', 'Whitelist');

-- 查看封鎖列表
SELECT * FROM ip_banned;
```

## 批量帳號管理

### 建立批量腳本

```bash
#!/bin/bash
# create_accounts.sh

ACCOUNTS=(
    "user1:pass1:user1@test.com"
    "user2:pass2:user2@test.com"
    "user3:pass3:user3@test.com"
)

for account in "${ACCOUNTS[@]}"; do
    IFS=':' read -r username password email <<< "$account"
    
    mysql -u acore -pacore_password acore_auth -e "
    INSERT INTO account (username, sha_pass_hash, email, expansion) 
    VALUES ('$username', SHA1(CONCAT(UPPER('$username'), ':', UPPER('$password'))), 
            '$email', 2);"
    
    echo "Created: $username"
done
```

## 進階管理功能

### 帳號資訊查詢

```
.account                 # 查看自己的帳號資訊
.pinfo [玩家名稱]       # 查看玩家資訊
.lookup player account [帳號名]  # 查詢帳號下的角色
```

### 帳號處罰

```
.ban account [帳號名] [時間] [原因]  # 封鎖帳號
.ban character [角色名] [時間] [原因]  # 封鎖角色
.ban ip [IP] [時間] [原因]  # 封鎖 IP
.unban account [帳號名]  # 解封帳號
```

## 實用 SQL 查詢

```sql
-- 查看所有 GM 帳號
SELECT username, gmlevel FROM account WHERE gmlevel > 0;

-- 查看最近登入
SELECT username, last_login, last_ip FROM account 
ORDER BY last_login DESC LIMIT 10;

-- 統計角色數量
SELECT a.username, COUNT(c.guid) as char_count 
FROM account a 
LEFT JOIN characters c ON a.id = c.account 
GROUP BY a.username;
```

## 下一步

帳號管理系統建立完成！下一篇將介紹客戶端連線設定。