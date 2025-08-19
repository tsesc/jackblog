+++
title = 'WoW 私服架設：伺服器核心配置與優化指南'
date = 2025-08-19T12:30:00+08:00
draft = false
tags = ['WoW', '伺服器配置', '優化', 'worldserver', 'authserver']
categories = ['WoW私服']
author = 'Jack'
description = '深入解析 worldserver.conf 和 authserver.conf 設定，優化伺服器效能與遊戲體驗'
toc = true
weight = 6
+++

## 前言

正確的伺服器配置能大幅提升遊戲體驗。本文將詳細介紹如何調整各項參數，打造最適合你的私服環境。

## Worldserver 核心設定

### 編輯 worldserver.conf

```bash
vim /opt/wow/server/etc/worldserver.conf
```

### 基本伺服器資訊

```conf
# 伺服器名稱與類型
RealmName = "我的私人伺服器"
RealmType = 0  # 0=PvE, 1=PvP, 6=RP, 8=RPPvP
RealmZone = 1  # 1=Development, 2=US, 3=Oceanic, 4=Latin, 5=Tournament

# 擴充版本
Expansion = 2  # 0=經典, 1=燃燒遠征, 2=巫妖王

# 語言設定
RealmLocale = 0  # 0=enUS, 4=zhCN, 5=zhTW
```

### 倍率設定

```conf
# 經驗值倍率
Rate.XP.Kill = 3.0
Rate.XP.Quest = 3.0
Rate.XP.Explore = 1.0

# 掉寶倍率
Rate.Drop.Item.Poor = 1.0
Rate.Drop.Item.Normal = 2.0
Rate.Drop.Item.Uncommon = 2.0
Rate.Drop.Item.Rare = 3.0
Rate.Drop.Item.Epic = 2.0
Rate.Drop.Item.Legendary = 1.0
Rate.Drop.Money = 2.0

# 聲望與榮譽
Rate.Reputation.Gain = 2.0
Rate.Honor = 2.0

# 技能熟練度
Rate.Skill.Discovery = 1.0
Rate.Skill.Gain.Crafting = 2.0
Rate.Skill.Gain.Gathering = 2.0
```

### 遊戲機制調整

```conf
# 死亡懲罰
Rate.Durability.Loss.Damage = 0.5
Rate.Durability.Loss.Death = 1.0
Death.SicknessLevel = 11  # 復活虛弱等級
Death.CorpseReclaimDelay = 0  # 撿屍體延遲

# 戰鬥設定
MaxPrimaryTradeSkill = 11  # 最大專業技能數
InstantFlightPaths = 1  # 即時飛行路徑
InstantLogout = 1  # 即時登出

# 跨陣營功能
AllowTwoSide.Interaction.Calendar = 1
AllowTwoSide.Interaction.Chat = 1
AllowTwoSide.Interaction.Channel = 1
AllowTwoSide.Interaction.Group = 1
AllowTwoSide.Interaction.Guild = 1
AllowTwoSide.Interaction.Trade = 1
AllowTwoSide.Interaction.Auction = 1
AllowTwoSide.Interaction.Mail = 1
```

### 副本與團隊設定

```conf
# 副本重置
Instance.ResetTimeHour = 4
Instance.UnloadDelay = 1800000  # 30分鐘

# 團隊副本
Rate.InstanceResetTime = 1.0

# 戰場設定
Battleground.CastDeserter = 0  # 不給逃兵懲罰
Battleground.QueueAnnouncer.Enable = 1
Battleground.QueueAnnouncer.PlayerOnly = 0

# 競技場
Arena.MaxRatingDifference = 500
Arena.RatingDiscardTimer = 600000  # 10分鐘
Arena.AutoDistributePoints = 0
```

## Authserver 設定

### 編輯 authserver.conf

```bash
vim /opt/wow/server/etc/authserver.conf
```

### 連線設定

```conf
# 監聽設定
BindIP = "0.0.0.0"
Port = 3724

# 連線限制
MaxPingTime = 30
RealmServerPort = 3724

# 安全設定
StrictVersionCheck = 0
WrongPass.MaxCount = 5
WrongPass.BanTime = 600
WrongPass.BanType = 0
```

## 效能優化

### CPU 優化

```conf
# 執行緒設定
UseProcessors = 0  # 0=使用所有核心
ProcessPriority = 0  # 0=正常優先級

# 地圖更新
MapUpdate.Threads = 4
MapUpdate.UpdateInterval = 100
```

### 記憶體優化

```conf
# 視野距離
Visibility.Distance.Continents = 90
Visibility.Distance.Instances = 120
Visibility.Distance.BGArenas = 180

# 生物視野
Visibility.Distance.Creature = 100
Visibility.Distance.Player = 100

# 網格載入
GridUnload = 1
GridUnloadDelay = 300000  # 5分鐘
```

### 網路優化

```conf
# 壓縮設定
Compression = 1
CompressionLevel = 1

# 封包限制
PacketSpoof.Policy = 1
PacketSpoof.BanMode = 0
```

## 特殊功能開關

### GM 功能

```conf
# GM 設定
GM.LoginState = 2  # GM登入狀態
GM.Visible = 1  # GM可見性
GM.Chat = 1  # GM聊天標記
GM.WhisperTo = 2  # 允許密語GM

# GM 指令
GM.LogTrade = 1
GM.StartLevel = 1
GM.AllowAchievementGain = 1
```

### 防作弊設定

```conf
# 反作弊
Anticheat.Enable = 1
Anticheat.ReportsForIngameSanctions = 70
Anticheat.MaxReportsForDailyReport = 70
Anticheat.BanTime = 86400  # 24小時

# Warden 反作弊
Warden.Enabled = 0  # 建議私服關閉
Warden.ClientResponseDelay = 600
Warden.ClientCheckHoldTime = 180
```

## 日誌設定

```conf
# 日誌等級
LogLevel = 1  # 0=最少, 3=最詳細
LogSQL = 0
LogColors = 1

# 日誌檔案
LogFile = "worldserver.log"
LogTimestamp = 1
LogFileLevel = 0

# 特殊日誌
DBErrorLogFile = "DBErrors.log"
CharLogFile = "Char.log"
ChatLogFile = "Chat.log"
RaLogFile = "RA.log"
```

## 自訂腳本設定

### 啟動公告

```conf
# 登入訊息
Motd = "歡迎來到私人伺服器！輸入 .help 查看指令。"

# 伺服器公告
Server.LoginInfo = 1
Announce.Broadcast.Timer = 600000  # 10分鐘
```

### 自動重啟

```bash
# 建立自動重啟腳本
vim /opt/wow/auto_restart.sh
```

```bash
#!/bin/bash
while true; do
    /opt/wow/server/bin/worldserver
    echo "Server crashed, restarting in 10 seconds..."
    sleep 10
done
```

## 常用調整建議

### 單人遊玩

```conf
Rate.XP.Kill = 7.0
Rate.Drop.Item.* = 3.0
Solocraft.Enable = 1
Solocraft.Difficulty = 5
```

### 小型私服（10-50人）

```conf
Rate.XP.Kill = 3.0
Rate.Drop.Item.* = 2.0
MaxPlayerLevel = 80
StartPlayerLevel = 1
```

### PvP 伺服器

```conf
RealmType = 1
Rate.Honor = 3.0
Battleground.Random.ResetHour = 6
Arena.SeasonID = 8
```

## 下一步

伺服器配置完成！你現在擁有：

✅ 自訂的遊戲體驗  
✅ 優化的效能設定  
✅ 完整的功能開關  
✅ 詳細的日誌系統  

下一篇將介紹遊戲帳號與權限管理。