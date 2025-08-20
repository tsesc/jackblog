+++
title = 'WoW 私服架設：進階模組安裝與自訂功能'
date = 2025-08-19T14:00:00+08:00
draft = false
tags = ['WoW', '模組', 'AzerothCore', '自訂功能', '進階設定']
categories = ['WoW私服']
author = 'Jack'
description = '介紹各種實用的 AzerothCore 模組安裝方法，包含跨陣營、全種族職業、拍賣場機器人等進階功能'
toc = true
weight = 109
+++

## 前言

AzerothCore 的模組系統讓你能輕鬆擴展伺服器功能。本文將介紹幾個熱門模組的安裝與設定方法。

## 模組安裝基礎

### 下載模組

```bash
cd /opt/wow/azerothcore-wotlk/modules
git clone [模組GitHub網址]
```

### 重新編譯

```bash
cd /opt/wow/azerothcore-wotlk
./acore.sh compiler clean
./acore.sh compiler all
```

## 熱門模組介紹

### 1. 無爐石冷卻時間

**模組名稱**: mod-no-hearthstone-cooldown

```bash
git clone https://github.com/azerothcore/mod-no-hearthstone-cooldown.git
```

設定檔：
```conf
# mod_no_hearthstone_cooldown.conf
NoHearthstoneCooldown.Enable = 1
```

### 2. 帳號通用功能

**模組名稱**: mod-account-wide

功能：
- 坐騎帳號通用
- 成就帳號通用
- 聲望帳號通用

```bash
git clone https://github.com/azerothcore/mod-account-mounts.git
```

### 3. 全種族全職業 (ARAC)

**模組名稱**: mod-arac

```bash
git clone https://github.com/azerothcore/mod-arac.git
```

設定：
```conf
# 允許所有種族選擇所有職業
ARAC.Enable = 1
```

### 4. 拍賣場機器人

**模組名稱**: mod-ah-bot

```bash
git clone https://github.com/azerothcore/mod-ah-bot.git
```

設定：
```conf
AuctionHouseBot.Enable = 1
AuctionHouseBot.Account = 1  # 機器人帳號ID
AuctionHouseBot.GUID = 1     # 機器人角色GUID

# 物品設定
AuctionHouseBot.Items.Amount.Grey = 0
AuctionHouseBot.Items.Amount.White = 2000
AuctionHouseBot.Items.Amount.Green = 5000
AuctionHouseBot.Items.Amount.Blue = 3000
AuctionHouseBot.Items.Amount.Purple = 1000
AuctionHouseBot.Items.Amount.Orange = 500
```

### 5. 單人副本縮放

**模組名稱**: mod-solocraft

```bash
git clone https://github.com/azerothcore/mod-solocraft.git
```

設定：
```conf
Solocraft.Enable = 1
Solocraft.Announce = 1
Solocraft.Difficulty = 5  # 難度倍數
```

### 6. 傳送大師 NPC

**模組名稱**: mod-npc-teleporter

```bash
git clone https://github.com/azerothcore/mod-npc-teleporter.git
```

生成 NPC：
```
.npc add 100000  # 傳送大師 NPC ID
```

### 7. 自動學習技能

**模組名稱**: mod-auto-learn-skills

功能：升級時自動學習職業技能

```bash
git clone https://github.com/azerothcore/mod-autolearn-skills.git
```

### 8. 跨陣營戰場

**模組名稱**: mod-crossfaction-battlegrounds

```bash
git clone https://github.com/azerothcore/mod-crossfaction-battlegrounds.git
```

設定：
```conf
CrossfactionBattlegrounds.Enable = 1
CrossfactionBattlegrounds.ShowPlayerName = 1
```

## 自訂功能開發

### 建立自訂腳本

```cpp
// 自訂歡迎腳本
class CustomWelcome : public PlayerScript
{
public:
    CustomWelcome() : PlayerScript("CustomWelcome") { }

    void OnLogin(Player* player) override
    {
        ChatHandler(player->GetSession()).SendSysMessage(
            "歡迎來到私人伺服器！"
        );
    }
};

void AddSC_custom_welcome()
{
    new CustomWelcome();
}
```

### 自訂物品

```sql
-- 添加自訂物品到資料庫
INSERT INTO item_template 
(entry, name, displayid, Quality, ItemLevel) 
VALUES 
(100000, '超級神劍', 50000, 5, 300);
```

### 自訂 NPC

```sql
-- 建立自訂 NPC
INSERT INTO creature_template 
(entry, name, subname, minlevel, maxlevel, faction) 
VALUES 
(100001, '新手指導員', '歡迎光臨', 80, 80, 35);
```

## 模組相容性檢查

### 測試步驟

1. 安裝模組後先編譯
2. 啟動伺服器檢查錯誤
3. 測試核心功能
4. 檢查日誌檔案

### 常見衝突

- 多個模組修改同一功能
- 資料庫結構衝突
- 版本不相容

## 效能影響評估

### 監控指標

```bash
# CPU 使用率
top -p $(pgrep worldserver)

# 記憶體使用
free -h

# 資料庫查詢
mysql -e "SHOW PROCESSLIST"
```

## 模組管理建議

1. **逐一安裝**：每次只安裝一個模組
2. **備份優先**：安裝前備份資料庫
3. **版本確認**：確保模組支援你的核心版本
4. **測試環境**：先在測試服測試

## 下一步

進階模組配置完成！最後一篇將介紹常見問題與疑難排解。