+++
title = 'WoW 私人伺服器 Docker 架設指南 (2025年版)'
date = 2025-08-22T10:00:00+08:00
draft = false
tags = ['WoW', '私人伺服器', 'AzerothCore', 'Playerbots', 'Docker', 'Windows', 'WSL2', '遊戲架設']
categories = ['WoW私服']
author = 'Jack'
description = '使用 Docker Compose 在 30 分鐘內架設完整的 WoW WotLK 3.3.5a 私人伺服器，包含 Playerbots 和完整配置'
toc = true
weight = 1
+++

使用 Docker 方式架設 WoW 私人伺服器，比傳統方式更簡單、更快速、更穩定。本指南將教你如何在 Windows 上使用 Docker Compose 建立完整的 AzerothCore + Playerbots 伺服器。

---

## 🚀 為什麼選擇 Docker 方式？

**主要參考來源:**
- Docker 架設教學影片: https://www.youtube.com/watch?v=Giv4UUg9CyI
- AzerothCore 官方文檔: https://www.azerothcore.org/wiki/faq

### Docker vs 傳統安裝方式對比

| 傳統方式 (VirtualBox + Linux) | Docker 方式 |
|---------------------------|------------|
| 需要安裝 VirtualBox | 只需 Docker Desktop |
| 需要安裝完整 Linux 系統 | 使用容器化環境 |
| 手動配置 MySQL | 自動配置資料庫 |
| 複雜的網路設定 | 簡化的網路管理 |
| 安裝時間：2-3 小時 | 安裝時間：30 分鐘 |
| 需要 Linux 知識 | 最少的技術門檻 |

### Docker 方式的優勢

✅ **極簡安裝** - 一個 docker-compose.yml 搞定所有服務  
✅ **完全隔離** - 不會影響主機系統  
✅ **易於備份** - 簡單的 volume 管理  
✅ **資源優化** - 比虛擬機更省資源  
✅ **快速重建** - 幾分鐘就能重新部署  
✅ **跨平台** - Windows/Mac/Linux 都能用  

## 🎮 為什麼要架設自己的 WoW 伺服器？

### 完全掌控的遊戲體驗

自架伺服器給您完全的控制權，包括：

- **自訂規則** - 修改經驗值倍率、掉寶率、金錢獲得
- **模組擴充** - 安裝數十種社群模組（塑形、帳號共享坐騎等）
- **Bot 系統** - 0-1600 個 AI 機器人讓世界充滿活力
- **跨陣營** - 允許聯盟部落組隊、公會、交易
- **永久保存** - 您的伺服器永遠屬於您，不會關服
- **免費開源** - 基於 AzerothCore 完全免費

### Playerbots 讓世界活起來

- 真實的 AI 玩家，有裝備、金錢、職業
- 自動任務、副本、戰場
- 可交易、組隊、指揮
- 讓單人遊戲也能體驗完整內容

## 📋 系統需求

### 硬體需求（Docker 版本）

| 項目 | 最低需求 | 建議配置 |
|-----|---------|---------|
| CPU | 雙核心 | 四核心以上 |
| 記憶體 | 8GB | 16GB |
| 硬碟空間 | 30GB | 50GB SSD |
| 作業系統 | Windows 10 | Windows 11 |

### 必要軟體

1. **Docker Desktop for Windows**
   - 下載：https://www.docker.com/products/docker-desktop/
   - 包含 WSL2 支援

2. **WoW 3.3.5a 客戶端**
   - ChromieCraft 版本（17GB）：https://www.chromiecraft.com/en/downloads/
   - 已修復相機 bug 版本：https://github.com/brndd/vanilla-tweaks/issues/17

3. **資料庫管理工具**（選用）
   - HeidiSQL：https://www.heidisql.com/download.php
   - 用於進階資料庫管理

4. **插件**
   - Unbot（控制機器人）：https://github.com/noisiver/unbot-addon/tree/english
   - TipTac（增強提示）：https://felbite.com/addon/4716-tiptac/
   - AI VoiceOver（語音朗讀）：https://github.com/mrthinger/wow-voiceover/

## 🐳 步驟一：安裝 Docker Desktop

### 1.1 下載並安裝 Docker Desktop

1. 前往 [Docker 官網](https://www.docker.com/products/docker-desktop/)
2. 下載 Windows 版本
3. 執行安裝程式，勾選：
   - ✅ Use WSL 2 instead of Hyper-V (推薦，效能更好)
   - ✅ Add shortcut to desktop

### 1.2 設定 WSL2

開啟 PowerShell（**非管理員身份**）執行：

```powershell
# 安裝 Debian 
wsl.exe --install Debian

# 進入 Debian 並設定使用者
wsl.exe -d Debian

# 在 Debian 中更新系統
sudo apt update && sudo apt full-upgrade -y

# 安裝必要工具
sudo apt install git bsdmainutils -y
```

### 1.3 Docker Desktop 設定

1. 開啟 Docker Desktop
2. 進入 Settings → Resources → WSL Integration
3. 開啟 Debian 的 Toggle
4. 點擊 Apply & Restart
5. 重新啟動電腦

### 1.4 驗證安裝

重新啟動後，開啟 Docker Desktop，確認右下角顯示 "Engine running"。

## 🚢 步驟二：使用 Docker Compose 部署伺服器

### 2.1 選擇配置方案

#### 方案 A：純淨 AzerothCore（無 Playerbots）

在 PowerShell 中執行：

```powershell
# 建立專案目錄
cd ~
mkdir azerothcore
cd azerothcore

# 使用 git 下載官方 docker-compose
git clone https://github.com/azerothcore/azerothcore-wotlk.git --depth 1
cd azerothcore-wotlk

# 建立並啟動容器
docker compose up -d --build
```

#### 方案 B：AzerothCore + Playerbots（推薦）

在 PowerShell 或 Linux 終端中執行：

```bash
# 建立專案目錄
cd ~
mkdir -p github  # 或使用 azerothcore
cd github

# 下載 Playerbots 版本
git clone https://github.com/liyunfan1223/azerothcore-wotlk.git --branch=Playerbot
cd azerothcore-wotlk

# 下載 Playerbots 模組
cd modules
git clone https://github.com/liyunfan1223/mod-playerbots.git --branch=master
cd ..

# 修改 docker-compose.yml 加入 playerbots 掛載
nano docker-compose.yml
# 或使用 vim/vi
```

找到 `ac-worldserver` 服務的 volumes 部分（約第 87-90 行），添加 playerbots 模組掛載：

```yaml
  ac-worldserver:
    <<: *ac-shared-conf
    <<: *ac-service-conf
    stdin_open: true
    tty: true
    command: ./acore.sh run-worldserver
    image: ${COMPOSE_DOCKER_IMAGE_TAG:-acore/ac-wotlk-worldserver:master}
    restart: unless-stopped
    volumes:
      - ${DOCKER_VOL_DATA_FOLDER:-./data}:/azerothcore/env/dist/data
      - ${DOCKER_VOL_ETC_FOLDER:-./env/dist/etc}:/azerothcore/env/dist/etc
      # 添加下面這行
      - ./modules/mod-playerbots/:/azerothcore/modules/mod-playerbots:ro
```

繼續執行：

```bash
# 第一次啟動，讓 Docker 生成設定檔
docker compose up -d

# 等待容器初始化（約 1-2 分鐘）
docker compose logs -f ac-worldserver

# 當看到 "World initialized" 或類似訊息後，按 Ctrl+C 停止查看日誌

# 現在設定檔已經生成，複製 Playerbots 設定
docker compose cp env/dist/etc/modules/playerbots.conf.dist ac-worldserver:/azerothcore/env/dist/etc/modules/playerbots.conf

# 或者如果設定檔在本地已生成
cp env/dist/etc/modules/playerbots.conf.dist env/dist/etc/modules/playerbots.conf

# 重啟服務讓設定生效
docker compose restart
```

**注意事項：**
- 設定檔 (`authserver.conf`, `worldserver.conf`) 會在首次啟動時自動生成
- Playerbots 設定檔需要手動複製
- 如果 `env/dist/etc` 目錄不存在，Docker 會在首次執行時創建它

**⏱️ 預計時間：**
- 純淨版：15-20 分鐘
- Playerbots 版：20-30 分鐘

### 2.2 從純淨版升級到 Playerbots 版本

如果你已經安裝了純淨版 AzerothCore，想要升級到包含 Playerbots 的版本，請按照以下步驟：

#### 步驟 1：備份現有資料

```bash
# 備份資料庫
docker exec ac-database mysqldump -uroot -ppassword --all-databases > backup_before_playerbots.sql

# 停止現有容器
docker compose down

# 備份整個資料夾（選用但建議）
cd ~/azerothcore
cp -r azerothcore-wotlk azerothcore-wotlk-backup
```

#### 步驟 2：切換到 Playerbots 版本

```bash
# 進入專案目錄
cd ~/azerothcore/azerothcore-wotlk

# 添加 Playerbots 遠端倉庫
git remote add playerbots https://github.com/liyunfan1223/azerothcore-wotlk.git

# 獲取 Playerbots 分支
git fetch playerbots

# 切換到 Playerbots 分支
git checkout -b Playerbot playerbots/Playerbot

# 下載 Playerbots 模組
cd modules
git clone https://github.com/liyunfan1223/mod-playerbots.git --branch=master
cd ..
```

#### 步驟 3：更新設定檔

```bash
# 複製 Playerbots 設定檔
cp env/dist/etc/modules/playerbots.conf.dist env/dist/etc/modules/playerbots.conf

# 編輯 docker-compose.yml
nano docker-compose.yml
```

在 `ac-worldserver` 服務的 volumes 部分（約第 90 行）添加：

```yaml
volumes:
  - ./config:/azerothcore/etc
  - ./data:/azerothcore/data
  - ./logs:/azerothcore/logs
  # 添加下面這行
  - ./modules/mod-playerbots/:/azerothcore/modules/mod-playerbots:ro
```

#### 步驟 4：重建並啟動

```bash
# 清理舊的建構緩存
docker compose down -v

# 重新建構並啟動
docker compose up -d --build

# 檢查日誌確認啟動成功
docker compose logs -f ac-worldserver
```

#### 步驟 5：驗證升級

```bash
# 進入 worldserver
docker attach ac-worldserver

# 檢查 Playerbots 模組是否載入
# 應該會看到 "Loading Module: mod-playerbots"

# 測試 bot 指令
bot add

# 安全離開（Ctrl+P, Ctrl+Q）
```

#### 常見問題處理

**問題：資料庫版本不匹配**
```bash
# 讓 Playerbots 自動更新資料庫結構
docker compose up -d
# 等待自動遷移完成（查看日誌）
docker compose logs -f ac-worldserver
```

**問題：想要回滾到純淨版**
```bash
# 停止服務
docker compose down

# 還原備份
cd ~/azerothcore
rm -rf azerothcore-wotlk
mv azerothcore-wotlk-backup azerothcore-wotlk

# 還原資料庫
docker compose up -d ac-database
docker exec -i ac-database mysql -uroot -ppassword < backup_before_playerbots.sql

# 重新啟動
docker compose up -d
```

#### 完全刪除純淨版並重新安裝

如果你想完全刪除純淨版 AzerothCore 並重新安裝 Playerbots 版本：

**步驟 1：停止並刪除所有容器**

```bash
# 進入專案目錄
cd ~/azerothcore/azerothcore-wotlk

# 停止所有容器
docker compose down

# 刪除所有容器、網路、volumes（完全清理）
docker compose down -v --remove-orphans
```

**步驟 2：刪除 Docker 映像**

```bash
# 列出 AzerothCore 相關映像
docker images | grep acore

# 刪除所有 AzerothCore 映像
docker rmi $(docker images | grep acore | awk '{print $3}') -f

# 或者手動刪除特定映像
docker rmi acore/ac-wotlk-worldserver:latest -f
docker rmi acore/ac-wotlk-authserver:latest -f
```

**步驟 3：刪除本地檔案**

```bash
# 返回上層目錄
cd ~/azerothcore

# 完全刪除舊的專案資料夾
rm -rf azerothcore-wotlk

# 清理任何備份檔案（選用）
rm -f backup*.sql
```

**步驟 4：清理 Docker 系統（選用但建議）**

```bash
# 清理未使用的容器、網路、映像和緩存
docker system prune -a --volumes

# 顯示清理後的空間
docker system df
```

**步驟 5：重新安裝 Playerbots 版本**

```bash
# 確保在正確的目錄
cd ~
mkdir -p github  # 或 azerothcore
cd github

# 全新下載 Playerbots 版本
git clone https://github.com/liyunfan1223/azerothcore-wotlk.git --branch=Playerbot
cd azerothcore-wotlk

# 下載 Playerbots 模組
cd modules
git clone https://github.com/liyunfan1223/mod-playerbots.git --branch=master
cd ..

# 編輯 docker-compose.yml
nano docker-compose.yml
```

在約第 87-90 行的 worldserver volumes 部分加入：
```yaml
volumes:
  - ${DOCKER_VOL_DATA_FOLDER:-./data}:/azerothcore/env/dist/data
  - ${DOCKER_VOL_ETC_FOLDER:-./env/dist/etc}:/azerothcore/env/dist/etc
  # 添加下面這行
  - ./modules/mod-playerbots/:/azerothcore/modules/mod-playerbots:ro
```

**步驟 6：建立並啟動新的服務**

```bash
# 第一次建立並啟動（會自動下載映像並生成設定檔）
docker compose up -d

# 等待初始化完成（檢查日誌）
docker compose logs -f ac-worldserver

# 當看到 "World initialized" 後，設定檔已生成
# 按 Ctrl+C 停止查看日誌

# 如果有 playerbots.conf.dist，複製它
if [ -f "env/dist/etc/modules/playerbots.conf.dist" ]; then
    cp env/dist/etc/modules/playerbots.conf.dist env/dist/etc/modules/playerbots.conf
fi

# 重啟以載入所有設定
docker compose restart

# 確認服務正常運行
docker compose ps
docker compose logs -f
```

**快速清理腳本**

創建一個 `clean-acore.sh` 腳本來一鍵清理：

```bash
#!/bin/bash
echo "⚠️ 警告：這將完全刪除 AzerothCore 和所有資料！"
read -p "確定要繼續嗎？(yes/no): " confirm

if [ "$confirm" = "yes" ]; then
    echo "停止容器..."
    cd ~/azerothcore/azerothcore-wotlk 2>/dev/null
    docker compose down -v --remove-orphans 2>/dev/null
    
    echo "刪除映像..."
    docker rmi $(docker images | grep acore | awk '{print $3}') -f 2>/dev/null
    
    echo "刪除檔案..."
    cd ~
    rm -rf ~/azerothcore
    
    echo "清理 Docker..."
    docker system prune -a --volumes -f
    
    echo "✅ 清理完成！"
else
    echo "❌ 操作已取消"
fi
```

使用方法：
```bash
chmod +x clean-acore.sh
./clean-acore.sh
```

### 2.3 設定 Realm 名稱和 IP

#### 設定 Realm 名稱

```bash
# 進入 MySQL 容器
docker exec -it ac-database mysql -uroot -p
# 密碼通常是 password

# 在 MySQL 中執行
USE acore_auth;
UPDATE realmlist SET name = 'My Realm Name' WHERE id = 1;
EXIT;
```

#### 本地遊玩（LAN）

```bash
# 獲取您的區域網路 IP
ipconfig

# 假設是 192.168.1.100，在 MySQL 中設定
docker exec -it ac-database mysql -uroot -p

USE acore_auth;
UPDATE realmlist SET address = '192.168.1.100' WHERE id = 1;
EXIT;
```

#### 網際網路遊玩（WAN）

```bash
# 獲取外部 IP
curl ipv4.icanhazip.com

# 在 MySQL 中設定外部 IP 或 DDNS
docker exec -it ac-database mysql -uroot -p

USE acore_auth;
UPDATE realmlist SET address = 'your.external.ip' WHERE id = 1;
EXIT;
```

**路由器端口轉發（僅網際網路需要）：**
- 3724 TCP (Auth Server)
- 8085 TCP (World Server)

## 🎮 步驟三：遊戲設定

### 3.1 創建 GM 帳號

```bash
# 進入 worldserver 容器
docker attach ac-worldserver

# 創建帳號（在容器內執行）
account create admin password
account set gmlevel admin 3 -1

# 安全離開容器（重要！）
# 按 Ctrl+P 然後 Ctrl+Q
# 不要按 Ctrl+C！會關閉服務！
```

### 3.2 設定客戶端連線

1. 找到 WoW 客戶端資料夾
2. 編輯 `Data\enUS\realmlist.wtf`（或對應語言資料夾）
3. 設定內容：

```
set realmlist 127.0.0.1    # 本機遊玩
# 或
set realmlist 192.168.1.100  # 區網 IP
# 或
set realmlist your.external.ip  # 外部 IP
```

### 3.3 登入遊戲

1. 啟動 wow.exe
2. 使用剛創建的帳號登入
3. 享受您的私人伺服器！

### 3.4 重要 GM 指令

作為 GM，您可以使用各種指令來管理伺服器。以下是最常用的指令：

#### 基本 GM 指令

```bash
# 傳送指令
.tele <location>           # 傳送到指定地點
.tele <player> <location>  # 傳送玩家到指定地點
.summon <player>           # 召喚玩家到你身邊
.appear <player>           # 傳送到玩家身邊

# 等級和經驗值
.levelup <level>           # 提升等級
.modify money <copper>     # 修改金錢（銅幣單位）
.modify honor <amount>     # 修改榮譽點數
.modify speed <value>      # 修改移動速度（1-10）

# 物品和裝備
.additem <item_id> [count] # 添加物品
.additemset <itemset_id>   # 添加套裝
.learn <spell_id>          # 學習技能/法術
.unlearn <spell_id>        # 忘記技能/法術

# 角色管理
.revive                    # 復活自己
.revive <player>           # 復活玩家
.kick <player>             # 踢出玩家
.ban character <name> <time> <reason>  # 封禁角色
.unban character <name>    # 解封角色

# 伺服器管理
.server info               # 顯示伺服器資訊
.server restart <seconds>  # 重啟伺服器倒計時
.announce <message>        # 全伺服器公告
.notify <message>          # 螢幕中央通知
```

#### Playerbots 專用指令

```bash
# Bot 管理
bot add                    # 添加隨機 bot 到隊伍
bot add <name>            # 添加指定 bot
bot remove                # 移除所有 bot
bot remove <name>         # 移除指定 bot

# Bot 控制
/w <botname> follow       # 讓 bot 跟隨
/w <botname> stay         # 讓 bot 停留
/w <botname> attack       # 讓 bot 攻擊目標
/w <botname> assist       # 讓 bot 協助你

# Bot 裝備和物品
/w <botname> e            # 裝備所有物品
/w <botname> equip auto   # 自動裝備最佳裝備
/w <botname> sell all     # 賣出所有垃圾
/w <botname> repair       # 修理裝備

# Bot 策略
/w <botname> co <strategy>  # 設定戰鬥策略
/w <botname> nc <strategy>  # 設定非戰鬥策略
/w <botname> dead <strategy> # 設定死亡策略
```

#### 實用 GM 宏命令

創建這些宏命令方便快速使用：

```lua
-- 快速滿級滿裝
/run SendChatMessage(".levelup 79", "SAY")
/run SendChatMessage(".learn all", "SAY")
/run SendChatMessage(".modify money 99999999", "SAY")

-- 獲得全部飛行點
/run SendChatMessage(".learn 34090", "SAY")  -- 專家騎術
/run SendChatMessage(".learn 34091", "SAY")  -- 大師騎術
/run SendChatMessage(".learn 54197", "SAY")  -- 寒冷飛行

-- 解鎖所有副本
/run for i=1,1000 do SendChatMessage(".instance unbind all", "SAY") end
```

**完整 GM 指令列表**：https://www.azerothcore.org/wiki/gm-commands

## ⚙️ 步驟四：Playerbots 配置（選用）

### 4.1 編輯 Playerbots 設定

創建一個批次檔 `edit-playerbots.bat`：

```batch
docker exec -it ac-worldserver nano /azerothcore/env/dist/etc/modules/playerbots.conf
```

### 4.2 推薦的 Playerbots 設定

```ini
# 基本設定
AiPlayerbot.RandomBotAutologin = 1
AiPlayerbot.MinRandomBots = 400
AiPlayerbot.MaxRandomBots = 500
AiPlayerbot.RandomBotMinLevel = 1
AiPlayerbot.RandomBotMaxLevel = 80

# Bot 行為
AiPlayerbot.RandomBotGroupNearby = 1
AiPlayerbot.RandomBotSayWithoutMaster = 0
AiPlayerbot.AutoDoQuests = 1
AiPlayerbot.RandomBotAutoJoinBG = 1

# 效能優化
PlayerbotsDatabase.WorkerThreads = 4
PlayerbotsDatabase.SynchThreads = 4

# Bot 裝備
AiPlayerbot.AutoEquipUpgradeLoot = 1
AiPlayerbot.EquipmentPersistence = 0
AiPlayerbot.EquipmentPersistenceLevel = 80
```

### 4.3 重置 Bot 資料庫

如果需要重置所有 Bot：

```bash
# 編輯設定檔，設定以下選項
AiPlayerbot.DeleteRandomBotAccounts = 1

# 重啟服務
docker compose restart

# 立即改回
AiPlayerbot.DeleteRandomBotAccounts = 0

# 再次重啟
docker compose restart
```

## 🔧 步驟五：進階配置

### 5.1 安裝拍賣行機器人（AH Bot）

```bash
# 進入模組目錄
cd ~/azerothcore/azerothcore-wotlk/modules

# 下載 AH Bot 模組
git clone https://github.com/azerothcore/mod-ah-bot --branch=master

# 重新建構
cd ..
docker compose up -d --build

# 創建 AH Bot 帳號
docker attach ac-worldserver
account create ahbot ahbot
# Ctrl+P, Ctrl+Q 離開

# 設定 AH Bot
docker exec -it ac-worldserver nano /azerothcore/env/dist/etc/modules/mod_ahbot.conf
```

### 5.2 安裝其他熱門模組

#### 無爐石冷卻時間

```bash
cd ~/azerothcore/azerothcore-wotlk/modules
git clone https://github.com/BytesGalore/mod-no-hearthstone-cooldown.git
cd ..
docker compose up -d --build
```

#### 帳號共享坐騎

```bash
cd ~/azerothcore/azerothcore-wotlk/modules
git clone https://github.com/azerothcore/mod-account-mounts
cd ..
docker compose up -d --build
```

### 5.3 World Server 推薦設定

編輯 worldserver.conf：

```ini
# PvP 設定（Playerbots 需要）
GameType = 1

# 跨陣營功能
AllowTwoSide.Accounts = 1
AllowTwoSide.Interaction.Chat = 1
AllowTwoSide.Interaction.Auction = 1

# 效能優化
MapUpdate.Threads = 2
PlayerSaveInterval = 20000

# 生活品質改善
InstantLogout = 0
MaxPrimaryTradeSkill = 11
MailDeliveryDelay = 0
```

## 🐳 Docker 管理指令

### 常用 Docker 指令

```bash
# 查看所有容器狀態
docker ps -a

# 查看即時日誌
docker compose logs -f

# 停止所有服務
docker compose down

# 啟動所有服務
docker compose up -d

# 重啟特定服務
docker compose restart ac-worldserver

# 進入容器執行指令
docker exec -it ac-worldserver bash

# 備份資料庫
docker exec ac-database mysqldump -uroot -ppassword --all-databases > backup.sql

# 還原資料庫
docker exec -i ac-database mysql -uroot -ppassword < backup.sql
```

### 建立管理腳本

創建 `server-control.bat`：

```batch
@echo off
echo WoW Server Control Panel
echo ========================
echo 1. Start Server
echo 2. Stop Server
echo 3. Restart Server
echo 4. View Logs
echo 5. Backup Database
echo 6. Enter World Console
echo 7. Exit

set /p choice="Enter your choice: "

if %choice%==1 docker compose up -d
if %choice%==2 docker compose down
if %choice%==3 docker compose restart
if %choice%==4 docker compose logs -f
if %choice%==5 docker exec ac-database mysqldump -uroot -ppassword --all-databases > backup_%date%.sql
if %choice%==6 docker attach ac-worldserver

pause
```

## 📊 監控和維護

### 資源使用監控

```bash
# 查看容器資源使用
docker stats

# 查看硬碟使用
docker system df

# 清理未使用的資源
docker system prune -a
```

### 定期維護

1. **每日**：檢查日誌是否有錯誤
2. **每週**：備份資料庫
3. **每月**：清理 Docker 緩存
4. **需要時**：更新 AzerothCore 和模組

### 更新伺服器

```bash
# 更新 AzerothCore
cd ~/azerothcore/azerothcore-wotlk
git pull

# 更新 Playerbots
cd modules/mod-playerbots
git pull

# 重新建構
cd ../..
docker compose up -d --build
```

## 🎯 常見問題解決

### 問題 1：容器無法啟動

```bash
# 檢查日誌
docker compose logs ac-worldserver

# 重置容器
docker compose down -v
docker compose up -d --build
```

### 問題 2：無法連線到伺服器

1. 檢查防火牆設定
2. 確認 realmlist.wtf 設定正確
3. 確認 Docker Desktop 正在運行

### 問題 3：Bot 不接受組隊邀請

```bash
# 重置 Bot 資料庫
docker exec -it ac-database mysql -uroot -ppassword

USE acore_characters;
DELETE FROM character_social WHERE flags = 1;
EXIT;
```

### 問題 4：記憶體使用過高

調整 Docker Desktop 設定：
1. Settings → Resources
2. 限制 Memory 為系統的 50-75%
3. 限制 CPU 為系統的 50-75%

## 🎉 結語

恭喜！您現在擁有了一個完整的 WoW 私人伺服器。使用 Docker 方式不僅簡化了安裝流程，也讓維護變得更加容易。

### 後續建議

1. **加入社群**：[AzerothCore Discord](https://discord.gg/azerothcore)
2. **探索模組**：[模組目錄](https://www.azerothcore.org/catalogue.html)
3. **優化設定**：根據需求調整 worldserver.conf 和 playerbots.conf
4. **定期備份**：使用提供的腳本定期備份資料

### 相關資源

- **影片教學**：https://www.youtube.com/watch?v=Giv4UUg9CyI
- **AzerothCore 官網**：https://www.azerothcore.org/
- **Playerbots Wiki**：https://github.com/liyunfan1223/mod-playerbots/wiki
- **ChromieCraft 下載**：https://www.chromiecraft.com/
- **詳細架設指南**：https://abs.freemyip.com:84/api/public/dl/ShUDo8u5?inline=true
- **GM 指令完整列表**：https://www.azerothcore.org/wiki/gm-commands

---

*本指南基於 2025 年最新的 Docker 部署方式編寫，相比傳統方式大幅簡化了安裝流程。如有問題歡迎在評論區討論。*