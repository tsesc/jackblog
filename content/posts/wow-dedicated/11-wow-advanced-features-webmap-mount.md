+++
title = 'WoW 私服進階功能：Azeroth Web Map 與自訂飛行坐騎'
date = 2025-08-19T15:00:00+08:00
draft = false
tags = ['WoW', 'Web Map', '飛行坐騎', '進階功能', 'PHP']
categories = ['WoW私服']
author = 'Jack'
description = '教你如何設定 Azeroth Web Map 即時顯示玩家位置，以及創建自訂的全地圖飛行坐騎'
toc = true
weight = 111
+++

## 前言

本文將介紹兩個超酷的進階功能：Web Map 能在瀏覽器上即時顯示所有玩家和機器人的位置，而自訂飛行坐騎則能讓你在任何地方飛行，包括原本禁飛的區域！

## 第一部分：Azeroth Web Map

### 什麼是 Azeroth Web Map？

這是一個基於 PHP 的網頁應用程式，能在地圖上顯示所有玩家和機器人的即時位置。雖然不是真正的即時更新（預設每 15 分鐘更新），但你可以調整更新頻率。

### 安裝前準備

#### 調整玩家位置儲存頻率

編輯 worldserver.conf：

```bash
world
# 修改儲存間隔（毫秒）
PlayerSaveInterval = 20000  # 每 20 秒儲存一次
```

### 安裝 Web Map

#### 步驟 1：安裝 PHP 和 Apache

```bash
apt update && apt install php php-mysqli apache2 -y
```

#### 步驟 2：下載 Web Map

```bash
cd /var/www/html
git clone https://github.com/DustinHendrickson/DustinsAzerothMap.git map
```

#### 步驟 3：設定自動重新導向

```bash
# 編輯 Apache 設定
nano /etc/apache2/apache2.conf

# 在檔案最底部加入
RedirectMatch ^/$ /map/

# 重啟 Apache
systemctl restart apache2.service
```

### 存取 Web Map

在瀏覽器輸入你的伺服器 IP：

```
http://192.168.1.250
```

你應該能看到艾澤拉斯地圖，上面標示著所有玩家的位置！

### 設定說明

Web Map 會自動連接到你的 MySQL 資料庫讀取玩家位置。如果無法顯示，請檢查：

1. Apache 服務是否運行
2. PHP 是否正確安裝
3. 資料庫連線是否正常
4. 玩家位置是否有更新（登出或等待 SaveInterval）

## 第二部分：自訂全地圖飛行坐騎

### 功能特色

- 等級需求：1 級（原本是 45 級）
- 可在任何地方飛行（包括東部王國、卡利姆多）
- 飛行速度：310%
- 獨特外觀

### 創建飛行坐騎

#### 步驟 1：停止伺服器

```bash
stop
```

#### 步驟 2：新增物品到資料庫

```bash
sudo mysql -u root
use acore_world

# 刪除舊版本（如果存在）
DELETE FROM `item_template` WHERE `entry`=701000;

# 插入新的飛行坐騎
INSERT INTO `item_template` (`entry`, `class`, `subclass`, `SoundOverrideSubclass`, `name`, `displayid`, `Quality`, `Flags`, `FlagsExtra`, `BuyCount`, `BuyPrice`, `SellPrice`, `InventoryType`, `AllowableClass`, `AllowableRace`, `ItemLevel`, `RequiredLevel`, `RequiredSkill`, `RequiredSkillRank`, `requiredspell`, `requiredhonorrank`, `RequiredCityRank`, `RequiredReputationFaction`, `RequiredReputationRank`, `maxcount`, `stackable`, `ContainerSlots`, `stat_type1`, `stat_value1`, `stat_type2`, `stat_value2`, `stat_type3`, `stat_value3`, `stat_type4`, `stat_value4`, `stat_type5`, `stat_value5`, `stat_type6`, `stat_value6`, `stat_type7`, `stat_value7`, `stat_type8`, `stat_value8`, `stat_type9`, `stat_value9`, `stat_type10`, `stat_value10`, `ScalingStatDistribution`, `ScalingStatValue`, `dmg_min1`, `dmg_max1`, `dmg_type1`, `dmg_min2`, `dmg_max2`, `dmg_type2`, `armor`, `holy_res`, `fire_res`, `nature_res`, `frost_res`, `shadow_res`, `arcane_res`, `delay`, `ammo_type`, `RangedModRange`, `spellid_1`, `spelltrigger_1`, `spellcharges_1`, `spellppmRate_1`, `spellcooldown_1`, `spellcategory_1`, `spellcategorycooldown_1`, `spellid_2`, `spelltrigger_2`, `spellcharges_2`, `spellppmRate_2`, `spellcooldown_2`, `spellcategory_2`, `spellcategorycooldown_2`, `spellid_3`, `spelltrigger_3`, `spellcharges_3`, `spellppmRate_3`, `spellcooldown_3`, `spellcategory_3`, `spellcategorycooldown_3`, `spellid_4`, `spelltrigger_4`, `spellcharges_4`, `spellppmRate_4`, `spellcooldown_4`, `spellcategory_4`, `spellcategorycooldown_4`, `spellid_5`, `spelltrigger_5`, `spellcharges_5`, `spellppmRate_5`, `spellcooldown_5`, `spellcategory_5`, `spellcategorycooldown_5`, `bonding`, `description`, `PageText`, `LanguageID`, `PageMaterial`, `startquest`, `lockid`, `Material`, `sheath`, `RandomProperty`, `RandomSuffix`, `block`, `itemset`, `MaxDurability`, `area`, `Map`, `BagFamily`, `TotemCategory`, `socketColor_1`, `socketContent_1`, `socketColor_2`, `socketContent_2`, `socketColor_3`, `socketContent_3`, `socketBonus`, `GemProperties`, `RequiredDisenchantSkill`, `ArmorDamageModifier`, `duration`, `ItemLimitCategory`, `HolidayId`, `ScriptName`, `DisenchantID`, `FoodType`, `minMoneyLoot`, `maxMoneyLoot`, `flagsCustom`, `VerifiedBuild`) 
VALUES (701000, 9, 0, -1, 'Tome of World Flying', 61330, 7, 134217792, 0, 1, 4500000, 4500000, 0, -1, -1, 80, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1000, 0, 0, 483, 0, -1, 0, -1, 0, -1, 31700, 6, 0, 0, -1, 0, -1, 0, 0, 0, 0, -1, 0, -1, 0, 0, 0, 0, -1, 0, -1, 0, 0, 0, 0, -1, 0, -1, 0, 'Learn to fly everywhere', 0, 0, 0, 0, 0, 8, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, -1, 0, 0, 0, 0, '', 0, 0, 0, 0, 0, 1);

exit;
```

#### 步驟 3：清除客戶端快取

**重要**：如果物品等級需求沒有更新，需要刪除快取：

```
刪除以下資料夾中的所有檔案：
C:\ChromieCraft_3.3.5a\Cache\WDB\enUS\
```

#### 步驟 4：啟動伺服器並取得坐騎

```bash
start
```

在遊戲中（必須是 GM）：
```
.additem 701000
```

學習後在坐騎視窗（Shift+P）就能看到新坐騎！

### 自訂坐騎屬性

你可以修改以下屬性：

- **RequiredLevel**: 需求等級（目前為 1）
- **BuyPrice/SellPrice**: 購買/出售價格
- **Quality**: 物品品質（7 = 神器）
- **spellid_1**: 法術 ID（483 是飛行技能）

## 進階配置

### Web Map 進階設定

#### 新增認證保護

如果你想限制 Web Map 的存取：

```bash
# 建立密碼檔案
htpasswd -c /etc/apache2/.htpasswd admin

# 編輯 Apache 設定
nano /var/www/html/map/.htaccess

# 加入以下內容
AuthType Basic
AuthName "Restricted Content"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

#### 自訂地圖樣式

編輯 `/var/www/html/map/config.php` 可以調整：
- 地圖縮放等級
- 玩家圖示大小
- 更新頻率
- 顏色設定

### 多個自訂坐騎

你可以創建多個不同的飛行坐騎：

```sql
-- 創建不同速度的坐騎
-- 入門級飛行坐騎（150% 速度）
INSERT INTO item_template (entry, name, spellid_1, RequiredLevel) 
VALUES (701001, 'Apprentice Flying Tome', 481, 1);

-- 專家級飛行坐騎（280% 速度）
INSERT INTO item_template (entry, name, spellid_1, RequiredLevel) 
VALUES (701002, 'Expert Flying Tome', 482, 1);
```

## 故障排除

### Web Map 無法顯示

1. 檢查 Apache 錯誤日誌：
```bash
tail -f /var/log/apache2/error.log
```

2. 確認 PHP 模組載入：
```bash
php -m | grep mysqli
```

3. 測試資料庫連線：
```php
<?php
$conn = new mysqli("localhost", "acore", "acore", "acore_characters");
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?>
```

### 坐騎無法飛行

1. 確認法術 ID 正確（483）
2. 檢查角色是否有飛行技能
3. 清除 WDB 快取
4. 重新登入角色

## 總結

這兩個功能大大提升了私服的遊戲體驗：

✅ Web Map 讓你隨時掌握伺服器動態  
✅ 自訂飛行坐騎打破原版限制  
✅ 完全自訂化的遊戲體驗  
✅ 簡單的安裝和設定流程  

現在你的私服不只功能完整，還有獨特的特色功能！