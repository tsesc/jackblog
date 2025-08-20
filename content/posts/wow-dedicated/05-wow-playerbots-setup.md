+++
title = 'WoW 私服架設：PlayerBots AI 機器人完整設定教學'
date = 2025-08-19T12:00:00+08:00
draft = false
tags = ['WoW', 'PlayerBots', 'AI', '機器人', '模組']
categories = ['WoW私服']
author = 'Jack'
description = '詳細介紹如何設定 PlayerBots 模組，讓你的私服充滿智慧型 AI 玩家，打造熱鬧的遊戲世界'
toc = true
weight = 105
+++

## 前言

PlayerBots 是讓私服變得生動的關鍵模組。它能添加 AI 控制的機器人玩家，讓你即使一個人也能體驗完整的團隊遊戲樂趣。

## PlayerBots 功能特色

- **智慧型 AI**：機器人會自動戰鬥、治療、施放增益
- **職業完整**：支援所有職業和專精
- **隊伍協作**：可組成 5 人小隊或 25 人團隊
- **自訂行為**：可調整機器人的戰鬥策略
- **隨機活動**：機器人會在世界中自由活動

## 設定檔配置

### 編輯 mod_playerbots.conf

```bash
vim /opt/wow/server/etc/mod_playerbots.conf
```

### 基本設定

```conf
# 啟用 PlayerBots
AiPlayerbot.Enabled = 1

# 機器人數量設定
AiPlayerbot.MinRandomBots = 50
AiPlayerbot.MaxRandomBots = 200
AiPlayerbot.RandomBotMinLevel = 1
AiPlayerbot.RandomBotMaxLevel = 80

# 機器人帳號前綴
AiPlayerbot.RandomBotAccountPrefix = "rndbot"
AiPlayerbot.RandomBotAccountCount = 200

# 地圖分布
AiPlayerbot.RandomBotMaps = "0,1,530,571"

# 起始區域機器人
AiPlayerbot.RandomBotStartingLevel = 5
AiPlayerbot.RandomBotsPerInterval = 10
AiPlayerbot.RandomBotsRefreshInterval = 60
```

### 進階行為設定

```conf
# 戰鬥行為
AiPlayerbot.BotFollowDistance = 50
AiPlayerbot.ReactDistance = 100
AiPlayerbot.SightDistance = 75
AiPlayerbot.SpellDistance = 30

# 自動接受
AiPlayerbot.AutoAcceptQuests = 1
AiPlayerbot.AutoPickReward = 1
AiPlayerbot.AutoEquipUpgradeLoot = 1

# 社交功能
AiPlayerbot.AllowGuildBots = 1
AiPlayerbot.RandomBotGuilds = "聯盟公會,部落公會"
AiPlayerbot.RandomBotJoinLfg = 1
AiPlayerbot.RandomBotJoinBG = 1

# 經濟系統
AiPlayerbot.RandomBotBuyItems = 1
AiPlayerbot.RandomBotSellItems = 1
AiPlayerbot.BotRepairWhenNeed = 1
```

### 職業特定設定

```conf
# 坦克職業
AiPlayerbot.ClassRaceProbability.1 = 20  # 戰士
AiPlayerbot.ClassRaceProbability.2 = 15  # 聖騎士
AiPlayerbot.ClassRaceProbability.6 = 10  # 死亡騎士

# 治療職業
AiPlayerbot.ClassRaceProbability.5 = 15  # 牧師
AiPlayerbot.ClassRaceProbability.7 = 10  # 薩滿
AiPlayerbot.ClassRaceProbability.11 = 10 # 德魯伊

# DPS 職業
AiPlayerbot.ClassRaceProbability.3 = 10  # 獵人
AiPlayerbot.ClassRaceProbability.4 = 10  # 盜賊
AiPlayerbot.ClassRaceProbability.8 = 15  # 法師
AiPlayerbot.ClassRaceProbability.9 = 10  # 術士
```

## 機器人管理指令

### 召喚機器人

在遊戲中使用以下指令：

```
.bot add [機器人名稱]     # 添加指定機器人
.bot add random           # 添加隨機機器人
.bot remove [機器人名稱]  # 移除機器人
.bot remove all          # 移除所有機器人
```

### 控制指令

```
/w [機器人名] follow      # 跟隨
/w [機器人名] stay        # 待命
/w [機器人名] attack      # 攻擊
/w [機器人名] flee        # 逃跑
/w [機器人名] tank        # 設為坦克
/w [機器人名] heal        # 設為治療
/w [機器人名] dps         # 設為 DPS
```

### 裝備管理

```
/w [機器人名] equip auto  # 自動裝備
/w [機器人名] sell gray   # 賣掉灰色物品
/w [機器人名] repair      # 修理裝備
/w [機器人名] talent      # 查看天賦
```

## 建立機器人帳號

### 批量建立腳本

```bash
vim /opt/wow/create_bot_accounts.sh
```

加入內容：

```bash
#!/bin/bash

MYSQL_USER="acore"
MYSQL_PASS="acore_password"
DB_NAME="acore_auth"

for i in {1..200}
do
    ACCOUNT="rndbot$i"
    PASSWORD="password$i"
    
    mysql -u$MYSQL_USER -p$MYSQL_PASS $DB_NAME -e "
    INSERT INTO account (username, sha_pass_hash, email, reg_mail, expansion) 
    VALUES ('$ACCOUNT', SHA1(CONCAT(UPPER('$ACCOUNT'), ':', UPPER('$PASSWORD'))), 
            'bot$i@server.local', 'bot$i@server.local', 2);"
    
    echo "Created account: $ACCOUNT"
done

echo "Bot accounts created successfully!"
```

執行腳本：
```bash
chmod +x /opt/wow/create_bot_accounts.sh
./create_bot_accounts.sh
```

## 優化機器人表現

### 資料庫優化

```sql
-- 優化機器人資料表
OPTIMIZE TABLE acore_characters.characters;
OPTIMIZE TABLE acore_characters.character_inventory;
OPTIMIZE TABLE acore_characters.character_spell;

-- 建立索引
CREATE INDEX idx_bot_account ON acore_auth.account(username);
CREATE INDEX idx_bot_guid ON acore_characters.characters(guid);
```

### 效能調整

```conf
# 減少 CPU 使用
AiPlayerbot.UpdateCycle = 100        # 更新週期（毫秒）
AiPlayerbot.MaxProcessTime = 50      # 最大處理時間
AiPlayerbot.ReactDelay = 500         # 反應延遲

# 記憶體優化
AiPlayerbot.CacheLevel = 2           # 快取等級
AiPlayerbot.LogLevel = 1             # 日誌等級（1=錯誤）
```

## 自訂機器人策略

### 策略控制指令

你可以使用遊戲內聊天（密語、隊伍、團隊頻道）來控制機器人的策略：

```
# 查看與修改策略
co ?        # 列出戰鬥策略 (combat strategies)
nc ?        # 列出非戰鬥策略 (non-combat strategies)

# 停止治療者浪費法力在 DPS 上
nc -dps assist    # 移除 DPS 輔助策略
nc ?              # 確認策略已移除
```

**注意**：如果重新初始化機器人，策略會重置為 playerbots.conf 中的預設值。

### 常用策略調整

#### 治療專注設定
讓治療者只專注治療，不進行攻擊：
```
/w [機器人名] nc -dps assist
/w [機器人名] nc +heal
```

#### 坦克策略
確保坦克維持仇恨：
```
/w [機器人名] co +tank
/w [機器人名] co +threat
```

#### DPS 優化
讓 DPS 更積極攻擊：
```
/w [機器人名] co +dps assist
/w [機器人名] co +aoe
```

### 建立策略檔案

```bash
vim /opt/wow/server/etc/aiplayerbot/strategies/custom.txt
```

範例策略：

```
# 坦克策略
tank:
  - defensive stance
  - shield block > 80% health
  - taunt on aggro loss
  - thunder clap > 3 enemies

# 治療策略  
healer:
  - heal tank < 50% health
  - group heal < 70% health
  - dispel debuffs
  - mana management

# DPS 策略
dps:
  - attack from behind
  - use cooldowns on boss
  - aoe > 3 enemies
  - focus priority target
```

## 機器人外觀設定

### 隨機外觀

```conf
# 外觀多樣性
AiPlayerbot.RandomBotRandomiseAppearance = 1
AiPlayerbot.RandomBotShowHelmet = 1
AiPlayerbot.RandomBotShowCloak = 1

# 裝備品質
AiPlayerbot.RandomGearQuality = "2,3,4"  # 綠、藍、紫
AiPlayerbot.RandomBotArmorQuality = 3
AiPlayerbot.RandomBotWeaponQuality = 3
```

## 故障排除

### 機器人不出現

```bash
# 檢查帳號是否建立
mysql -u acore -p acore_auth -e "SELECT * FROM account WHERE username LIKE 'rndbot%';"

# 檢查模組載入
tail -f /opt/wow/server/logs/worldserver.log | grep PlayerBot
```

### 機器人不動作

```conf
# 調整設定
AiPlayerbot.AllowActivity = all
AiPlayerbot.BotActiveAlone = 1
AiPlayerbot.RandomBotUpdateInterval = 10
```

## 實用技巧

### 組建完美隊伍

```
# 5 人小隊配置
.bot add warrior    # 坦克
.bot add priest     # 治療
.bot add mage       # DPS
.bot add hunter     # DPS
.bot add rogue      # DPS
```

### 副本攻略

```
# 設定隊伍模式
/w all formation arrow
/w tank protect healer
/w dps assist tank
/w healer focus tank
```

## 下一步

PlayerBots 設定完成！現在你的私服擁有：

✅ 智慧型 AI 機器人  
✅ 豐富的互動體驗  
✅ 自訂戰鬥策略  
✅ 完整的隊伍系統  

下一篇將介紹伺服器的基本配置調整。