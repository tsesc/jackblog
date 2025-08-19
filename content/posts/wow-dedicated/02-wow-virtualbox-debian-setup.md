+++
title = 'WoW 私服架設：VirtualBox 與 Debian 系統安裝詳解'
date = 2025-08-19T10:30:00+08:00
draft = false
tags = ['WoW', 'VirtualBox', 'Debian', 'Linux', '虛擬機']
categories = ['WoW私服']
author = 'Jack'
description = '詳細說明如何安裝設定 VirtualBox 虛擬機與 Debian Linux 系統，為 WoW 私服建立穩定的運行環境'
toc = true
weight = 2
+++

## 前言

在上一篇文章中，我們了解了架設 WoW 私服所需的軟硬體需求。現在，讓我們開始實際操作，首先建立虛擬機環境並安裝 Debian Linux 系統。

## 第一部分：安裝 VirtualBox

### 下載與安裝

1. 前往 [VirtualBox 官網](https://www.virtualbox.org/wiki/Downloads)
2. 選擇 Windows 版本下載
3. 執行安裝程式，使用預設設定即可
4. 安裝完成後重新啟動電腦

### 安裝擴充套件

VirtualBox Extension Pack 提供額外功能支援：

1. 下載 Extension Pack（與 VirtualBox 版本相同）
2. 雙擊 .vbox-extpack 檔案
3. 按照提示完成安裝

## 第二部分：建立虛擬機

### 步驟 1：新增虛擬機

1. 開啟 VirtualBox，點擊「新增」
2. 設定虛擬機基本資訊：
   - **名稱**：WoW-Server
   - **類型**：Linux
   - **版本**：Debian (64-bit)

### 步驟 2：配置硬體資源

#### 記憶體配置
```
建議配置：4096 MB (4GB)
最低配置：2048 MB (2GB)
```

#### 硬碟設定
1. 選擇「立即建立虛擬硬碟」
2. 硬碟檔案類型：VDI
3. 儲存方式：動態配置
4. 硬碟大小：50GB

### 步驟 3：進階設定

在虛擬機設定中調整以下項目：

#### 系統設定
- **主機板**：
  - 開機順序：硬碟、光碟
  - 晶片組：PIIX3
  - 啟用 I/O APIC

- **處理器**：
  - CPU 數量：2-4 核心
  - 執行上限：100%
  - 啟用 PAE/NX

#### 顯示設定
- 視訊記憶體：128MB
- 加速：啟用 3D 加速

#### 網路設定
```
配接卡 1：NAT（預設）
配接卡 2：僅限主機介面卡（可選）
```

## 第三部分：安裝 Debian Linux

### 準備安裝媒體

1. 下載 [Debian 12 (Bookworm) ISO](https://www.debian.org/download)
2. 在 VirtualBox 中掛載 ISO：
   - 選擇虛擬機 → 設定 → 存放裝置
   - 點擊光碟圖示 → 選擇 ISO 檔案

### 開始安裝

#### 啟動虛擬機
1. 點擊「啟動」按鈕
2. 選擇 "Install" 或 "Graphical Install"

#### 基本設定
1. **語言選擇**：English（建議）或繁體中文
2. **地區**：Taiwan
3. **鍵盤配置**：美式英文

#### 網路設定
1. **主機名稱**：wow-server
2. **網域名稱**：留空或設定為 local

#### 使用者帳號
1. **Root 密碼**：設定強密碼（記住它！）
2. **一般使用者**：
   - 全名：WoW Admin
   - 使用者名稱：wowadmin
   - 密碼：設定密碼

### 磁碟分割

#### 簡易設定（建議新手）
1. 選擇「引導 - 使用整個磁碟」
2. 選擇虛擬硬碟
3. 選擇「所有檔案在同一分割區」
4. 確認並寫入變更

#### 進階分割（選用）
```
/boot    - 500MB  (ext4)
/        - 20GB   (ext4)
/home    - 15GB   (ext4)
swap     - 4GB
/var     - 剩餘空間 (ext4)
```

### 軟體選擇

在軟體選擇畫面，只勾選以下項目：
- SSH server
- 標準系統工具

**不要**安裝桌面環境，以節省資源。

### 完成安裝

1. 安裝 GRUB 開機載入器到主要磁碟
2. 移除安裝媒體（退出 ISO）
3. 重新啟動系統

## 第四部分：初始系統設定

### 登入系統

使用 root 帳號登入：
```bash
Debian GNU/Linux 12 wow-server tty1
wow-server login: root
Password: [輸入 root 密碼]
```

### 更新系統

```bash
# 更新套件清單
apt update

# 升級已安裝套件
apt upgrade -y

# 安裝必要工具
apt install -y sudo vim curl wget git net-tools
```

### 設定靜態 IP（選用）

編輯網路設定檔：
```bash
vim /etc/network/interfaces
```

加入以下內容：
```
auto enp0s3
iface enp0s3 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

重啟網路服務：
```bash
systemctl restart networking
```

### 設定 SSH 存取

#### 允許 root SSH 登入（開發環境）
```bash
# 編輯 SSH 設定
vim /etc/ssh/sshd_config

# 找到並修改
PermitRootLogin yes

# 重啟 SSH 服務
systemctl restart sshd
```

#### 查看虛擬機 IP
```bash
ip addr show
```

記下 IP 位址，稍後使用 PuTTY 連線。

## 第五部分：使用 PuTTY 連線

### 安裝 PuTTY

1. 下載 [PuTTY](https://www.putty.org/)
2. 安裝到 Windows 系統

### 設定連線

1. 開啟 PuTTY
2. 輸入連線資訊：
   - Host Name：虛擬機 IP
   - Port：22
   - Connection type：SSH

3. 儲存 Session：
   - Session name：WoW-Server
   - 點擊 Save

### 首次連線

1. 點擊 Open 開始連線
2. 接受主機金鑰
3. 使用 root 帳號登入

## 第六部分：系統優化

### 設定時區

```bash
# 設定為台北時區
timedatectl set-timezone Asia/Taipei

# 確認時區
date
```

### 設定 Swap

如果安裝時未設定 swap，可手動建立：

```bash
# 建立 4GB swap 檔案
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

# 永久啟用
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### 安裝開發工具

```bash
# 安裝編譯工具
apt install -y build-essential cmake make

# 安裝其他必要套件
apt install -y libssl-dev libmysqlclient-dev
apt install -y libreadline-dev zlib1g-dev
apt install -y libbz2-dev libboost-all-dev
```

## 第七部分：建立快照

在繼續之前，建立虛擬機快照以便回復：

1. 關閉虛擬機：`shutdown -h now`
2. 在 VirtualBox 中選擇虛擬機
3. 點擊「快照」→「建立」
4. 命名：「Debian 基礎系統安裝完成」

## 故障排除

### 網路連線問題

如果無法連線到網際網路：
```bash
# 檢查網路介面
ip link show

# 啟用網路介面
ip link set enp0s3 up

# 檢查路由
ip route show
```

### SSH 連線失敗

1. 確認 SSH 服務運行中：
```bash
systemctl status sshd
```

2. 檢查防火牆（Debian 預設無防火牆）

3. 確認 IP 位址正確

### 虛擬機效能問題

- 增加配置的 CPU 核心數
- 增加記憶體配置
- 啟用虛擬化技術（VT-x/AMD-V）

## 下一步

恭喜！你已經成功建立了 WoW 私服的基礎環境。我們現在有了：

✅ 運行中的 VirtualBox 虛擬機  
✅ 安裝完成的 Debian Linux 系統  
✅ SSH 遠端連線能力  
✅ 基本的系統工具  

在下一篇文章中，我們將開始安裝 AzerothCore 核心，這是私服的心臟部分。

## 小提示

- 定期建立快照，方便回復
- 記錄所有密碼在安全的地方
- 保持系統更新
- 熟悉基本 Linux 指令會很有幫助

## Debian 13 TRIXIE 特殊安裝說明

如果你使用的是 Debian 13 TRIXIE，需要進行以下額外步驟：

### 修改套件庫來源

```bash
# 編輯套件來源檔案
nano /etc/apt/sources.list

# 將以下兩行中的 "trixie" 改為 "sid"
# 原始：
deb http://deb.debian.org/debian/ trixie main non-free-firmware
deb-src http://deb.debian.org/debian/ trixie main non-free-firmware

# 修改為：
deb http://deb.debian.org/debian/ sid main non-free-firmware
deb-src http://deb.debian.org/debian/ sid main non-free-firmware
```

### 安裝 MySQL

```bash
# 更新套件並安裝 MySQL
apt update && apt-get install -y mysql-server libmysqlclient-dev mysql-client
```

### 修改 AzerothCore 安裝腳本

在執行 `git clone` 之後，需要修改安裝腳本：

```bash
# 編輯 Debian 安裝腳本
nano ~/azerothcore-wotlk/apps/installer/includes/os_configs/debian.sh

# 註解掉以下幾行（在前面加上 #）：
# run noninteractive install for MYSQL 8.4 LTS
#wget https://dev.mysql.com/get/mysql-apt-config_0.8.32-1_all.deb -P "$VAR_PATH"
#DEBIAN_FRONTEND="noninteractive" $SUDO dpkg -i "$VAR_PATH/mysql-apt-config_0.8.32-1_all.deb"
#$SUDO apt-get update
#DEBIAN_FRONTEND="noninteractive" $SUDO apt-get install -y mysql-server libmysqlclient-dev
```

完成這些步驟後，就可以繼續正常的安裝流程了。

準備好進入核心安裝了嗎？讓我們繼續前進！