+++
title = 'WoW 私服架設：客戶端連線設定與插件安裝'
date = 2025-08-19T13:30:00+08:00
draft = false
tags = ['WoW', '客戶端', 'realmlist', '插件', '連線設定']
categories = ['WoW私服']
author = 'Jack'
description = '完整說明如何設定 WoW 客戶端連線到私服，包含 realmlist 修改、插件安裝與常見連線問題解決'
toc = true
weight = 8
+++

## 前言

客戶端設定是連接私服的最後一步。本文將詳細介紹如何修改客戶端連線設定、安裝必要插件，以及解決常見連線問題。

## 客戶端版本確認

### 檢查版本

確保你的客戶端是 WoTLK 3.3.5a (Build 12340)：

1. 執行 Wow.exe
2. 在登入畫面左下角查看版本號
3. 必須顯示 3.3.5.12340

## 重要安全修補程式

### 修復相機/滑鼠錯誤

這個修補程式修復了相機突然跳轉到不同方向的問題：

**下載連結**：
- [GitHub 討論頁面](https://github.com/brndd/vanilla-tweaks/issues/17)
- [Mega 下載](https://mega.nz/folder/ykZglR6J#LOG-hGTqJ3ZOTROwTlG5Hw)

### RCE 漏洞修補

**重要**：3.3.5a 客戶端存在遠端執行程式碼（RCE）漏洞，私服擁有者可能在你的電腦上執行任意程式碼。

**修補程式**：
- [RCEPatcher GitHub](https://github.com/stoneharry/RCEPatcher/)
- 下載後執行，選擇你的 WoW.exe 檔案進行修補

### 安裝步驟

1. 備份原始 WoW.exe
2. 下載並執行 RCEPatcher
3. 選擇 WoW.exe 進行修補
4. 驗證修補成功

## 修改 Realmlist

### 找到 realmlist.wtf

檔案位置：
```
WoW客戶端目錄/Data/enUS/realmlist.wtf
或
WoW客戶端目錄/Data/zhTW/realmlist.wtf
```

### 編輯連線設定

使用記事本開啟，修改為：

```
set realmlist 你的伺服器IP
```

範例：
```
set realmlist 192.168.1.100  # 區域網路
set realmlist 127.0.0.1       # 本機測試
set realmlist your.domain.com # 網域名稱
```

## 防火牆設定

### Windows 防火牆

開啟以下連接埠：
- 3724 (authserver)
- 8085 (worldserver)

### 路由器 Port Forwarding

如果要公開伺服器：
```
外部埠口 3724 -> 內部 IP:3724
外部埠口 8085 -> 內部 IP:8085
```

## 必要插件安裝

### Unbot 插件

用於顯示機器人資訊：

1. 下載 Unbot 插件
2. 解壓縮到 `Interface/AddOns/`
3. 重新啟動客戶端

### TipTac 插件

增強提示框架功能：

1. 下載 TipTac for 3.3.5a
2. 解壓縮到 `Interface/AddOns/`
3. 在角色選擇畫面啟用

## 客戶端優化

### Config.wtf 設定

編輯 `WTF/Config.wtf`：

```
SET locale "zhTW"
SET realmName "你的伺服器名稱"
SET gameTip "0"
SET accounttype "LK"
SET groundEffectDensity "256"
SET groundEffectDist "140"
SET smallCull "1"
SET skycloudlod "3"
SET characterAmbient "1"
SET extshadows "1"
SET shadowmode "3"
SET showfootprintparticles "1"
SET showfootprints "1"
SET showshadow "1"
```

## 首次連線

### 登入步驟

1. 執行 Wow.exe
2. 輸入帳號密碼
3. 選擇伺服器（Realm）
4. 建立角色
5. 進入遊戲

### 測試連線

在遊戲中按 `Ctrl+R` 查看延遲：
- 綠色（< 100ms）：優秀
- 黃色（100-300ms）：良好
- 紅色（> 300ms）：需要優化

## 常見問題解決

### 無法連線到伺服器

1. 檢查 realmlist.wtf 設定
2. 確認伺服器 IP 正確
3. 測試網路連線：
```cmd
ping 伺服器IP
telnet 伺服器IP 3724
```

### 卡在角色選擇畫面

1. 檢查 worldserver 是否運行
2. 查看伺服器日誌
3. 清除客戶端快取：
```
刪除 Cache 資料夾
刪除 WTF 資料夾（備份設定）
```

### 版本不相容

錯誤訊息：「版本不正確」

解決方法：
1. 確認客戶端版本為 3.3.5a
2. 檢查 authserver.conf 的版本設定
3. 下載正確版本的客戶端

## 多開設定

### 允許多個客戶端

編輯 worldserver.conf：
```conf
MaxPlayerLevel = 80
MaxInstancesPerHour = 10
```

### 批次啟動腳本

建立 `LaunchWoW.bat`：
```batch
@echo off
start "" "C:\WoW\Wow.exe"
timeout /t 5
start "" "C:\WoW\Wow.exe"
```

## 客戶端模組推薦

### 必裝插件
- DBM (Deadly Boss Mods)
- Recount (傷害統計)
- Bagnon (背包整合)
- Bartender4 (動作條)

### 輔助插件
- QuestHelper (任務助手)
- AtlasLoot (副本掉落)
- Auctioneer (拍賣場)

## 下一步

客戶端設定完成！下一篇將介紹進階模組安裝。