+++
title = 'WoW 私人伺服器架設指南：概述與準備工作'
date = 2025-08-19T10:00:00+08:00
draft = false
tags = ['WoW', '私服', 'AzerothCore', '遊戲伺服器', 'Linux']
categories = ['WoW私服']
author = 'Jack'
description = '完整介紹如何架設自己的魔獸世界巫妖王之怒私人伺服器，包含所需軟硬體規格與前置準備'
toc = true
weight = 101
+++

## 前言

你是否曾經想過擁有一個完全屬於自己的魔獸世界伺服器？一個可以自由修改規則、添加機器人玩家，甚至可以跨陣營組隊的世界？本系列文章將帶你一步步架設自己的《魔獸世界：巫妖王之怒》（WoTLK 3.3.5a）私人伺服器。

## 為什麼選擇 AzerothCore？

AzerothCore 是目前最活躍的開源魔獸世界模擬器專案之一，具有以下優勢：

- **開源免費**：完全開放原始碼，可自由修改
- **活躍社群**：持續更新與錯誤修復
- **豐富模組**：支援各種功能擴充
- **穩定性高**：經過長期測試與優化
- **PlayerBots 支援**：可添加 AI 控制的機器人玩家

## 系統架構概述

我們將使用以下架構來建立私人伺服器：

```
Windows 主機
    └── VirtualBox 虛擬機
        └── Debian Linux
            ├── AzerothCore 核心
            ├── MySQL 資料庫
            └── PlayerBots 模組
```

這種架構的好處是：
- 保持 Windows 環境不受影響
- 易於備份與還原
- 資源隔離，避免系統衝突

## 硬體需求

### 最低配置
- **處理器**：四核心 CPU（Intel i5 或 AMD Ryzen 5）
- **記憶體**：8GB RAM（分配 4GB 給虛擬機）
- **硬碟空間**：50GB 可用空間
- **網路**：穩定的網路連線

### 建議配置
- **處理器**：六核心以上 CPU
- **記憶體**：16GB RAM（分配 8GB 給虛擬機）
- **硬碟空間**：100GB SSD 空間
- **網路**：有線網路連線

## 軟體需求

### 主機端（Windows）
1. **作業系統**：Windows 7/10/11
2. **虛擬化軟體**：[VirtualBox](https://www.virtualbox.org/) 7.0 或更新版本
3. **SSH 客戶端**：[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
4. **資料庫管理工具**：[HeidiSQL](https://www.heidisql.com/)
5. **遊戲客戶端**：WoTLK 3.3.5a 版本

### 虛擬機端（Linux）
1. **作業系統**：[Debian 12 (Bookworm)](https://www.debian.org/)
2. **資料庫**：MariaDB 10.11
3. **編譯工具**：GCC、CMake、Make

## 前置準備工作

### 1. 下載必要檔案

在開始之前，請先下載以下檔案：

- VirtualBox 安裝程式
- Debian 12 ISO 映像檔（約 650MB）
- PuTTY 安裝程式
- HeidiSQL 安裝程式

### 2. 取得遊戲客戶端

你需要一個 WoTLK 3.3.5a 版本的遊戲客戶端。這個版本發布於 2010 年，是巫妖王之怒資料片的最終版本。

### 3. 規劃網路設定

決定你的伺服器網路配置：
- **本機測試**：使用 NAT 網路即可
- **區域網路**：需要設定橋接網路
- **公開伺服器**：需要設定 Port Forwarding

## 伺服器功能預覽

完成架設後，你的私人伺服器將具備以下功能：

### 基本功能
- 完整的巫妖王之怒遊戲內容
- 自訂經驗值倍率
- 自訂掉寶率
- GM 管理員權限

### PlayerBots 功能
- AI 控制的機器人玩家
- 可邀請機器人組隊
- 機器人自動戰鬥
- 模擬真實玩家行為

### 可選模組
- 無爐石冷卻時間
- 帳號通用坐騎
- 全種族全職業（ARAC）
- 拍賣場機器人
- 跨陣營功能

## 時間規劃

整個架設過程預計需要：

1. **環境準備**：30 分鐘
2. **系統安裝**：45 分鐘
3. **核心編譯**：60-90 分鐘
4. **基本設定**：30 分鐘
5. **測試調整**：30 分鐘

總計約 3-4 小時即可完成基本架設。

## 注意事項

### 法律聲明
- 本教學僅供個人學習與研究使用
- 不得用於商業營利目的
- 需遵守相關智慧財產權法規

### 技術提醒
- 定期備份資料庫與設定檔
- 注意防火牆設定
- 保護好管理員帳號密碼
- 監控系統資源使用狀況

## 下一步

準備好所有軟硬體後，我們將在下一篇文章中開始安裝 VirtualBox 與 Debian 系統。這將是我們伺服器的基礎環境。

整個過程看似複雜，但只要按部就班，你很快就能擁有屬於自己的魔獸世界！

## 相關資源

- [AzerothCore 官方網站](https://www.azerothcore.org/)
- [AzerothCore GitHub](https://github.com/azerothcore/azerothcore-wotlk)
- [PlayerBots 模組](https://github.com/liyunfan1223/mod-playerbots)
- [AzerothCore Wiki](https://www.azerothcore.org/wiki)
- [WoW 私服架設資源](https://abs.freemyip.com:84/api/public/dl/ShUDo8u5?inline=true)

準備好開始你的私服冒險了嗎？讓我們開始吧！