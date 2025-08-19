+++
title = 'WoW 私服架設：AzerothCore 核心安裝與編譯完整教學'
date = 2025-08-19T11:00:00+08:00
draft = false
tags = ['WoW', 'AzerothCore', '編譯', 'C++', 'CMake']
categories = ['WoW私服']
author = 'Jack'
description = '詳細說明如何下載、編譯和安裝 AzerothCore with PlayerBots，建立功能完整的 WoW 私服核心'
toc = true
weight = 3
+++

## 前言

在前面的文章中，我們已經準備好了 Debian Linux 環境。現在要進入最關鍵的步驟：安裝 AzerothCore 核心。這是整個私服的心臟，負責處理所有遊戲邏輯。

## 什麼是 AzerothCore？

AzerothCore 是一個開源的魔獸世界伺服器模擬器，特點包括：

- 基於 TrinityCore 和 MaNGOS 發展而來
- 支援 WoTLK 3.3.5a 版本
- 模組化架構，易於擴展
- 活躍的開發社群
- 優秀的效能和穩定性

## 第一步：安裝依賴套件

### 基本編譯工具

透過 SSH 連線到伺服器，執行以下命令：

```bash
# 更新套件清單
apt update && apt upgrade -y

# 安裝基本編譯工具
apt install -y git cmake make gcc g++ clang

# 安裝必要的函式庫
apt install -y libmysqlclient-dev libssl-dev libbz2-dev
apt install -y libreadline-dev libncurses-dev libboost-all-dev
apt install -y mysql-server libace-dev
```

### 安裝額外工具

```bash
# 安裝額外的有用工具
apt install -y screen tmux htop curl wget unzip
apt install -y p7zip-full gdb
```

## 第二步：下載 AzerothCore 原始碼

### 克隆 PlayerBots 分支

我們將使用包含 PlayerBots 的特殊分支：

```bash
# 建立工作目錄
mkdir -p /opt/wow
cd /opt/wow

# 克隆 AzerothCore with PlayerBots
git clone https://github.com/liyunfan1223/azerothcore-wotlk.git \
    --branch=Playerbot --depth=1
    
# 進入目錄
cd azerothcore-wotlk
```

### 下載 PlayerBots 模組

```bash
# 進入模組目錄
cd modules

# 克隆 PlayerBots 模組
git clone https://github.com/liyunfan1223/mod-playerbots.git \
    --branch=master --depth=1

# 返回主目錄
cd ..
```

## 第三步：使用自動化腳本安裝

AzerothCore 提供了便利的自動化腳本：

### 安裝所有依賴

```bash
# 執行依賴安裝腳本
./acore.sh install-deps
```

這個腳本會自動：
- 檢查系統環境
- 安裝缺少的依賴
- 設定必要的環境變數

### 編譯核心

```bash
# 使用全部 CPU 核心編譯
./acore.sh compiler all

# 或指定核心數（例如使用 4 核心）
./acore.sh compiler all -j 4
```

編譯過程需要 60-90 分鐘，取決於系統性能。

## 第四步：手動編譯方法（可選）

如果自動腳本失敗，可以手動編譯：

### 建立編譯目錄

```bash
# 建立並進入編譯目錄
mkdir build
cd build
```

### 配置 CMake

```bash
# 配置編譯選項
cmake ../ \
    -DCMAKE_INSTALL_PREFIX=/opt/wow/server \
    -DCMAKE_C_COMPILER=/usr/bin/clang \
    -DCMAKE_CXX_COMPILER=/usr/bin/clang++ \
    -DWITH_WARNINGS=1 \
    -DTOOLS=1 \
    -DSCRIPTS=static \
    -DMODULES=static
```

### 開始編譯

```bash
# 編譯（使用所有 CPU 核心）
make -j $(nproc)

# 安裝編譯完成的檔案
make install
```

## 第五步：下載遊戲資料檔

### 取得客戶端資料

AzerothCore 需要從遊戲客戶端提取的資料檔：

```bash
# 建立資料目錄
mkdir -p /opt/wow/server/data

# 下載預先提取的資料檔（約 1.5GB）
cd /opt/wow/server/data

# 方法一：使用官方提供的資料
wget https://github.com/wowgaming/client-data/releases/download/v16/data.zip
unzip data.zip
rm data.zip
```

### 資料檔結構

確認以下目錄存在：
```
/opt/wow/server/data/
├── dbc/
├── maps/
├── mmaps/
├── vmaps/
├── cameras/
└── gt/
```

## 第六步：設定目錄結構

### 建立必要目錄

```bash
# 建立日誌目錄
mkdir -p /opt/wow/server/logs

# 建立設定檔目錄（如果不存在）
mkdir -p /opt/wow/server/etc

# 設定權限
chmod -R 755 /opt/wow/server
```

### 複製設定檔

```bash
# 複製預設設定檔
cd /opt/wow/server/etc
cp authserver.conf.dist authserver.conf
cp worldserver.conf.dist worldserver.conf

# 如果有 PlayerBots 設定
cp mod_playerbots.conf.dist mod_playerbots.conf
```

## 第七步：建立啟動腳本

### 建立伺服器控制腳本

```bash
# 建立腳本檔案
vim /opt/wow/server/start.sh
```

加入以下內容：

```bash
#!/bin/bash

SERVER_PATH="/opt/wow/server"
SCREEN_NAME="wow"

case "$1" in
    start)
        echo "Starting WoW Server..."
        cd $SERVER_PATH/bin
        screen -dmS ${SCREEN_NAME}-auth ./authserver
        sleep 3
        screen -dmS ${SCREEN_NAME}-world ./worldserver
        echo "Server started in screen sessions."
        ;;
    stop)
        echo "Stopping WoW Server..."
        screen -X -S ${SCREEN_NAME}-auth quit
        screen -X -S ${SCREEN_NAME}-world quit
        echo "Server stopped."
        ;;
    restart)
        $0 stop
        sleep 5
        $0 start
        ;;
    status)
        screen -ls | grep $SCREEN_NAME
        ;;
    attach-world)
        screen -r ${SCREEN_NAME}-world
        ;;
    attach-auth)
        screen -r ${SCREEN_NAME}-auth
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status|attach-world|attach-auth}"
        exit 1
        ;;
esac
```

設定執行權限：
```bash
chmod +x /opt/wow/server/start.sh
```

## 第八步：驗證安裝

### 檢查編譯結果

```bash
# 檢查執行檔是否存在
ls -la /opt/wow/server/bin/

# 應該看到以下檔案：
# authserver
# worldserver
# 以及其他工具程式
```

### 測試執行檔

```bash
# 測試 authserver
/opt/wow/server/bin/authserver --version

# 測試 worldserver
/opt/wow/server/bin/worldserver --version
```

## 常見編譯問題

### 問題 1：記憶體不足

如果編譯時出現記憶體錯誤：
```bash
# 建立或增加 swap
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

# 使用較少的平行編譯
make -j 2  # 只使用 2 個核心
```

### 問題 2：缺少依賴

```bash
# 重新執行依賴安裝
./acore.sh install-deps

# 或手動安裝特定套件
apt-get install -y libmysqlclient-dev
```

### 問題 3：Git 子模組問題

```bash
# 更新所有子模組
git submodule update --init --recursive
```

## 編譯優化建議

### 使用 Clang 編譯器

Clang 通常比 GCC 編譯速度更快：
```bash
# 安裝 Clang
apt install -y clang

# 在 CMake 時指定
-DCMAKE_C_COMPILER=clang
-DCMAKE_CXX_COMPILER=clang++
```

### 使用 ccache 加速

```bash
# 安裝 ccache
apt install -y ccache

# 設定環境變數
export CC="ccache gcc"
export CXX="ccache g++"
```

## 第九步：初次執行準備

在首次啟動伺服器前，需要：

1. 設定資料庫（下一篇文章詳述）
2. 配置伺服器設定檔
3. 導入初始資料
4. 建立管理員帳號

## 下一步

恭喜！AzerothCore 核心已經編譯完成。我們現在擁有：

✅ 編譯完成的 authserver 和 worldserver  
✅ 必要的遊戲資料檔  
✅ 基本的目錄結構  
✅ 伺服器控制腳本  

下一篇文章將介紹如何設定 MySQL 資料庫，這是存儲所有遊戲資料的地方。

## 小提示

- 編譯過程可能需要 1-2 小時，請耐心等待
- 建議在編譯完成後建立虛擬機快照
- 保留原始碼目錄，方便未來更新
- 定期更新到最新版本以獲得錯誤修復

## 有用的命令

```bash
# 查看編譯進度
tail -f build/CMakeFiles/CMakeOutput.log

# 監控系統資源
htop

# 查看 screen 會話
screen -ls

# 清理編譯檔案（如需重新編譯）
cd build && make clean
```

準備好設定資料庫了嗎？讓我們繼續前進！