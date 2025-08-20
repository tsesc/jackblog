+++
title = 'WoW 私人伺服器完整架設指南 (2025年版) - 原始文檔'
date = 2025-08-20T10:00:00+08:00
draft = false
tags = ['WoW', '私人伺服器', 'AzerothCore', 'Playerbots', 'Linux', 'Debian', '遊戲架設', '教學文檔']
categories = ['WoW私服']
author = 'Jack'
description = '完整的 WoW WotLK 3.3.5a 私人伺服器架設指南，包含 AzerothCore + Playerbots 的詳細安裝步驟和配置說明'
toc = true
weight = 112
+++

本文為完整的 WoW 私人伺服器架設原始技術文檔，保留所有原始格式和內容。

---

## How to run your own WotLK AzerothCore with Playerbots server using Linux (2025)

**更新紀錄:**
- **2025-08-15**: Debian 13 TRIXIE 安裝說明已加入文末！另外重新整理了設定 realm 名稱和 LAN/INTERNET 連線的部分
- **2025-08-15 Update2**: 展示如何設定 AZEROTH WEB MAP 顯示 BOT 位置！
- **2025-08-15 Update3**: 修正 Part 6 - 創建可在任何地方使用的飛行坐騎！等級需求降至 1 級！
- **2025-08-16 Update4**: 新增 ah 別名快速進入拍賣行機器人設定檔

**影片教學系列:**
- Part 1 Installation: https://www.youtube.com/watch?v=DwJ6OfPophw
- Part 2 Bot Control: https://www.youtube.com/watch?v=ZGn5BxQeZSw
- Part 3 Installing modules: https://www.youtube.com/watch?v=DnHGuZlmdsM
- Part 4 Installing ARAC module - All Races All Classes! Undead Hunters!: https://www.youtube.com/watch?v=JjQ-tnk3C2o
- Part 5 Auction House Bot module! https://www.youtube.com/watch?v=ZSbvbFAcqzI

Today I am going to teach you how to run your own World of Warcraft: Wrath of the Lich King server from start to finish using
Virtualbox and Debian Linux.
My video is not going to be flashy, but I assure you that I am teaching you just about the best way to do this.
There are other guides out there on websites and youtube that recommend you run this in Windows using so-called "repacks" but
as a gamer since 1984, and someone who has ran his own dedicated game servers as far back as the Counterstrike beta, I strongly
disagree with that way of doing it. Don't just take my word for it - the server we are using - Azerothcore (which is free and open source on github) - agrees with me in their FAQ: https://www.azerothcore.org/wiki/faq

Q. Do you support Repacks based on AzerothCore?
A. No. Repacks are NOT supported and we strongly suggest to not use them for several reasons. You can check this tutorial for an easy way of installing AC without using any repack.
There are a lot of guides out there, but so far I've not seen any of them show you how to install the version with bots to make your server come to life, let alone basic server upkeep, bot control,
and additional cool things such as the AI voice addon which has narration for the quest text!

## Benefits to running your own WoW server

Retail World of Warcraft as we all know is a non-stop train - it keeps going and never stops. Private servers, on the other hand, are hosted
by fans across the world, but they all seem to eventually go offline for one reason or another. They also mostly beg you for money, or are pay to win, and are
generally for-profit. Azerothcore is completely different. Finally, a group of people decided to collaberate and allow anyone to submit changes and improvements with
their free and open source model! Although you can use this exact source code to run your own server, we will be using a slightly modified version that allows for bot integration.
The official Azerothcore website is here: https://www.azerothcore.org/
All of the other links are below. I am not a programmer and I have no affiliation with any of the tools we will be using today - I am just a gamer, a fan of World of Warcraft, and a strong supporter of freedom and open source software.

I will try not to get too technical in this tutorial. You should be able to get it up and running as quickly as possible. I could explain every little thing that is going on, but this
tutorial would be several hours long. We can run now and learn later! I tend to be more of a gamer than a knob turner! Self-hosting World of Warcraft gives you god-like
powers in terms of customization. Here are just some of the control you have in the game:

-Name your realm, assign GMs (game masters), create accounts (and what expansion that account has unlocked, from vanilla to WotLK), and more!
-Install/uninstall any of the dozens of fan-made modules for Azerothcore, such as transmog NPC, skip DK starting zone, account-wide mounts, and more!
-Allow cross-faction grouping, guilding, communication, and raiding. You can even decide to allow all classes for any race, such as a human druid! It's up to you.
-Change bot behavior, item drop rates, gold, boss health, mob health, mob spawn rates and locations.
-Change and/or disable cooldowns of abilities, debuffs, buffs, and everything else.
-Disable the daze effect (being hit from behind), hearth cooldown, hunter pet happiness, deserter debuff, and everything else! There is almost no limit!
-Keep up to date on future changes of playerbots and Azerothcore or decide to never update and your server will remain static and unchanged! A peaceful, relaxing azeroth that is always available.
-Open the server to remote connections and allow your friends anywhere in the world to connect to you! Or keep it private so only you or those on your home network to connect and play!
-The good feeling that you own your server and it cannot be taken away from you. You own it. Nobody will ask you for money, and Blizzard cannot take your server down. It is really yours.

Why would I want to play with bots?
-Bots make your sever feel alive, even if you're the only human logged into your realm!
-Bots will help you quest, dungeon, and raid. They will heal you, resurrect you,buff you, and even greet you!
-You can decide to have 0 bots, 1 bot, or 1600 bots running around on your realm. It's up to you.
-"Playerbots" are unique in that they are actual, valid accounts generated on your server. They have real inventories, gear, stats, and gold.
Unlike "NPCBots," Playerbots can be inspected, traded with, and commanded on a deeper level. For example, you can spawn a Dwarf priest to give yourself the Fear Ward buff!
-You can immediately change the spec and gear of your bots in your group at any time. You can even summon them to you wherever you are, even in a dungeon, without a warlock!
I have not tried it, but I'm fairly sure you can summon a real warlock to you and have the warlock cast an actual summoning stone, just like the real thing.

## Why use my guide?

I played World of Warcraft from January 2005 to sometime in 2021, mostly as "Painbow-Mal'Ganis," an Orc Hunter. I want to play
the game as accurately as possible without cheating, but I also want the ability to try interesting things out if I feel like it such as flying to Hyjal (a place you couldn't get to in 3.3.5a),
or give myself quality of life features such as instant log out. I have very high standards when it comes to running software on a computer, and spend hours seeking and self-testing
various methods until I am confident my methods are the most reliable, free, secure, robust, repeatable, responsive, and unobtrusive methods available.

## Hardware requirements for the server

50GB of hard drive space dedicated to the server
At least a quad core CPU (You could try with less. Not recommended.)
At least 4GB of dedicated memory to the VM (Virtual Machine)

## Software requirements

Windows 7, 10, 11
VirtualBox - https://www.virtualbox.org/wiki/Downloads
Debian Bookworm ISO - http://mirror.cogentco.com/debian-cd/current/amd64/iso-cd/
PuTTY (to connect to Debian and issue commands) (get 64-bit x86 version) - https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
PuTTY (to connect to Debian and issue commands) mirror if above link isn't working: https://abs.freemyip.com:84/share/kd-NUJeO
HeidiSQL (for easy database modifications) - https://www.heidisql.com/download.php?download=installer
WoTLK 3.3.5a client (to play the game) 17GB - https://www.chromiecraft.com/en/downloads/
Unbot addon (to control the bots in-game.) - https://github.com/noisiver/unbot-addon/tree/english
Unbot mirror link if the one above doesn't work: https://abs.freemyip.com:84/share/FD_vJrpt
TipTac tooltip addon: https://felbite.com/addon/4716-tiptac/
TipTac mirror link if above doesn't work: https://abs.freemyip.com:84/api/public/dl/29mVUeau

Optional (although recommended) software:
VoiceOver addon for World of Warcraft WotLK 3.3.5a: https://github.com/mrthinger/wow-voiceover/releases/download/v1.4.3/AI_VoiceOver-WoW_3.3.5-v1.4.3.zip
VoiceOver vanilla sounds: https://github.com/mrthinger/wow-voiceover/releases/download/v1.3.1/AI_VoiceOverData_Vanilla-v1.0.0.zip

2024-11-16 -
Patched clients that fixes the camera/mouse bug (the one that causes your camera to suddenly jerk in a totally different direction).
I HIGHLY recommend this fix!!
https://github.com/brndd/vanilla-tweaks/issues/17
Download mirror 1: https://mega.nz/folder/ykZglR6J#LOG-hGTqJ3ZOTROwTlG5Hw
Download mirror 2: https://abs.freemyip.com:84/share/Ij6DObNS

2024-11-16 - Important client patch that fixes an exploit. You can patch your custom clients as well!
In the 3.3.5a WoW client there is a Remote Code Exploit (RCE) that allows any private server owner to inject and run arbitrary code on your computer.
This patcher will modify your WoW executable file to fix the exploit.
Download it here.
https://github.com/stoneharry/RCEPatcher/
Download mirror: https://abs.freemyip.com:84/share/NFXfR-wT


## Getting started

You can do this even if you know nothing of programming, coding, Linux, operating systems, or anything real complex. You just have to be able to follow my
instructions exactly. I feel everyone - no matter the skill level - deserves to be able to enjoy running their own server.

### 基本安裝步驟

1. Install Virtualbox with all the default settings.
2. Install Debian Linux into Virtualbox
3. Install putty with default settings. See video guide for all my improvements and settings to all of these.

After installing Virtualbox, Debian, and Putty, we will now use putty to connect to the newly created Debian Linux PC using the username you created during Debian install.
Start by logging into the username you typed in during the Debian install. Let's say it was "nirv" it will appear like this after logging in:

nirv@azeroth:~$

From this point, let's make a password for the root account.
Root is the "administrator" account. This password can be the same or different from your user you created.

### 設定 root 密碼

```bash
# Change root password
sudo passwd
# don't make the password too large. we'll be typing it frequently

# Now let's log into the root account
su -
```

Logging into the root account will make your prompt look slightly different than above.

root@azeroth:~#

### SSH 和開機設定

```bash
# Allow logging in SSH as root directly (you can only SSH into user accounts if you do not change this)
sed -ie '0,/#PermitRootLogin prohibit-password/s/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && service sshd restart

# Faster boot
nano /etc/default/grub
```

編輯 GRUB 設定：
```
GRUB_DEFAULT=1
GRUB_TIMEOUT=0
```

```bash
update-grub
```

## Force your Debian Linux to keep the same IP address

### 查找網路資訊

```bash
# Find your current IP
ip a
# Let's assume it's 192.168.1.250. We will now force debian to keep that IP.

# Find the gateway. usually it's 192.168.1.1
apt install net-tools -y
route -n

# Find the network interface. It's the one that says "UP" here when you use this command. usually it's enp0s3 or something. Let's assume that's it.
ip -br -c link show

# With the current IP, the gateway, and the network interface, let's modify the config.
nano /etc/network/interfaces
```

### 設定靜態 IP

編輯網路設定檔，確保註解掉或刪除 "the primary network interface" 下方的 "allow-hotplug" 和 "iface" 設定：

```
# The primary network interface
#allow-hotplug enp0s3
#iface enp0s3 inet dhcp
auto enp0s3
iface enp0s3 inet static
 address 192.168.1.250
 netmask 255.255.255.0
 gateway 192.168.1.1
 dns-domain azeroth.core
 dns-nameservers 192.168.1.1
```

**注意：** 如果有連線問題，請確保將 "gateway" 和 "dns-nameservers" 設定為您的閘道器 IP（可能不是 192.168.1.1）

CTRL+O 儲存後輸入 exit 關閉 putty。在 Virtualbox 視窗中執行：

```bash
sudo systemctl restart networking.service
```

## We will now be installing the required software

### Container 特別注意事項 (2025-08-02)

如果要在容器（如 Proxmox）而非虛擬機中安裝 Azerothcore，在執行 git clone 後、install-deps 前需要修改設定：

```bash
# assuming you are using a Debian container, open this config.sh file
# and you will see this at about the 23rd line: # OSDISTRO="ubuntu"
# this will not work until you have ran the git clone command below first
nano ~/azerothcore-wotlk/conf/dist/config.sh
```

修改為：
```
OSDISTRO="debian"
```

### 主要安裝步驟

```bash
apt update && apt upgrade -y
apt update && apt install git curl unzip sudo libreadline-dev -y
git clone https://github.com/liyunfan1223/azerothcore-wotlk.git --branch=Playerbot
```

**⚠️ DEBIAN 13 用戶注意：** 如果您使用 DEBIAN 13 TRIXIE，請先閱讀本指南底部的「2025-08-15 INSTALLING IN DEBIAN 13 TRIXIE」章節！

```bash
cd ~/azerothcore-wotlk/modules
git clone https://github.com/liyunfan1223/mod-playerbots.git --branch=master

# DEPENDENCIES FOR DEBIAN!!
apt-get update && apt-get install git cmake make gcc g++ clang libssl-dev libbz2-dev libreadline-dev libncurses-dev libboost-all-dev tmux gnupg -y

cd ~/azerothcore-wotlk
./acore.sh install-deps
./acore.sh compiler all
```

### MySQL 遠端連線設定

允許遠端連線 MySQL（這裡的遠端指本地網路，不是網際網路。建議啟用以便使用 Windows 的 HeidiSQL 管理資料庫）：

```bash
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

找到 "bind-address" 和 "mysqlx-bind-address" 並替換（若找不到則貼在最底部）：

```
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
disable_log_bin
```

```bash
sudo systemctl restart mysql
```

### SQL Database configuration

創建 "acore" MySQL 用戶：

```bash
sudo mysql -u root
```

```sql
DROP USER IF EXISTS 'acore'@'localhost';
CREATE USER 'acore'@'localhost' IDENTIFIED BY 'acore' WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0;
GRANT ALL PRIVILEGES ON * . * TO 'acore'@'localhost' WITH GRANT OPTION;
CREATE DATABASE IF NOT EXISTS `acore_world` DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE IF NOT EXISTS `acore_characters` DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE IF NOT EXISTS `acore_auth` DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;
GRANT ALL PRIVILEGES ON `acore_world` . * TO 'acore'@'localhost' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON `acore_characters` . * TO 'acore'@'localhost' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON `acore_auth` . * TO 'acore'@'localhost' WITH GRANT OPTION;
exit;
```

繼續設定：

```bash
cd ~/azerothcore-wotlk
./acore.sh client-data
cp env/dist/etc/authserver.conf.dist env/dist/etc/authserver.conf
cp env/dist/etc/worldserver.conf.dist env/dist/etc/worldserver.conf
cp ~/azerothcore-wotlk/env/dist/etc/modules/playerbots.conf.dist ~/azerothcore-wotlk/env/dist/etc/modules/playerbots.conf
```

### 創建啟動腳本

現在我們要首次啟動伺服器並創建資料庫。首先創建啟動腳本：

```bash
nano /root/start.sh
```

將以下內容貼入檔案（CTRL+O 儲存）：

```bash
cd ~/azerothcore-wotlk/env/dist/bin
authserver="./authserver"
worldserver="./worldserver"

authserver_session="auth-session"
worldserver_session="world-session"

if tmux new-session -d -s $authserver_session; then
    echo "Created authserver session: $authserver_session"
else
    echo "Error when trying to create authserver session: $authserver_session"
fi

if tmux new-session -d -s $worldserver_session; then
    echo "Created worldserver session: $worldserver_session"
else
    echo "Error when trying to create worldserver session: $worldserver_session"
fi

if tmux send-keys -t $authserver_session "$authserver" C-m; then
    echo "Executed \"$authserver\" inside $authserver_session"
    echo "You can attach to $authserver_session and check the result using \"tmux attach -t $authserver_session\""
else
    echo "Error when executing \"$authserver\" inside $authserver_session"
fi

if tmux send-keys -t $worldserver_session "$worldserver" C-m; then
    echo "Executed \"$worldserver\" inside $worldserver_session"
    echo "You can attach to $worldserver_session and check the result using \"tmux attach -t $worldserver_session\""
else
    echo "Error when executing \"$worldserver\" inside $worldserver_session"
fi
```

### 創建快捷指令別名

```bash
nano ~/.bashrc
```

捲動到檔案最底部，貼上以下內容：

```bash
#START
alias wow='cd ~/azerothcore-wotlk;tmux attach -t world-session'
alias auth='cd ~/azerothcore-wotlk;tmux attach -t auth-session'
alias start='bash /root/start.sh'
alias stop='tmux kill-server'
alias compile='cd ~/azerothcore-wotlk;./acore.sh compiler all'
alias build='cd ~/azerothcore-wotlk;./acore.sh compiler build'
alias update='cd ~/azerothcore-wotlk;git pull;cd ~/azerothcore-wotlk/modules/mod-playerbots;git pull'
alias pb='nano ~/azerothcore-wotlk/env/dist/etc/modules/playerbots.conf'
alias world='nano ~/azerothcore-wotlk/env/dist/etc/worldserver.conf'
alias updatemods="cd ~/azerothcore-wotlk/modules;find . -mindepth 1 -maxdepth 1 -type d -print -exec git -C {} pull \;"
alias ah='nano ~/azerothcore-wotlk/env/dist/etc/modules/mod_ahbot.conf'
#END
```

CTRL+O 儲存，CTRL+X 離開編輯器，然後更新 bashrc：

```bash
source ~/.bashrc
```

### 啟動伺服器

```bash
# Now start your server by just typing start.
start

# Type wow to get into the world-server session where we can issue commands.
# On your first run, it will ask if you want to create your database. Type yes and wait.
wow
```

**提示：** 要離開 "world session" 畫面（底部有綠色條），按 CTRL+B 然後按 D 從 tmux session "detach"。如果覺得太麻煩，也可以直接關閉 putty 終端機。

```bash
# And you can see the auth server any time it's running by typing.
# This isn't really necessary for most cases and you shouldn't need to go in here.
auth
```


## SET YOUR REALM NAME

可以隨時更改：

```bash
sudo mysql -u root
```

```sql
use acore_auth;
UPDATE realmlist SET name = 'My Realm Name' WHERE id = 1;
exit;
```

## SET YOUR SERVER FOR LAN PLAY ONLY

可以隨時更改。請使用您自己的 IP 位址：

### LOCAL ONLY / LAN / YOUR NETWORK ONLY / NO INTERNET ACCESS / SINGLE PLAYER WITH BOTS

```bash
# Make sure to put your Linux IP here instead of mine!
# To get your LAN IP, type in terminal: ip a
sudo mysql -u root
```

```sql
use acore_auth;
UPDATE realmlist SET address = '192.168.1.250' WHERE id = 1;
exit;
```

## SET YOUR SERVER FOR INTERNET PLAY

可以隨時更改。請使用您自己的 IP 位址：

### INTERNET / TCP/IP / MULTIPLAYER / ALLOW YOUR FRIENDS TO CONNECT / WAN / OPEN TO INTERNET

您需要設定外部 IP 才能讓人連線。這是必須的，否則無人能連線！

```bash
# To easily obtain your external ip, type this in terminal
curl ipv4.icanhazip.com
# OR
curl ifconfig.me

# Then enter that IP here. THIS IS JUST A SAMPLE. PUT YOUR EXTERNAL IP HERE INSTEAD
sudo mysql -u root
```

```sql
use acore_auth;
UPDATE realmlist SET address = '66.24.137.208' WHERE id = 1;
exit;
```

或者如果您有 DDNS：

```bash
sudo mysql -u root
```

```sql
use acore_auth;
UPDATE realmlist SET address = 'dns.bobsaget.com' WHERE id = 1;
exit;
```

## ROUTER PORTS FOR REMOTE CONNECTIONS (PORT FORWARDING)

**僅網際網路 / TCP/IP / 多人遊戲需要，區域網路不需要**

如果要讓朋友連線到您的伺服器，必須在路由器開啟以下兩個連接埠：

```
3724 TCP AUTH
8085 TCP WORLD
```

將這些連接埠轉發到您的 WotLK 伺服器 IP 位址。

## OPTIONAL REMOTE SQL DATABASE CONNECTION (HeidiSQL)

如果您想使用 GUI 探索/修改 SQL 資料庫，可以設定遠端連線：

```bash
sudo mysql -u root
```

```sql
# replace the two "nirv" and the "banana" password below with the login/password you would like.
# This will be entered into your HeidiSQL software in Windows.
CREATE USER IF NOT EXISTS 'nirv'@'%' IDENTIFIED BY 'banana';
GRANT ALL PRIVILEGES ON *.* TO 'nirv'@'%' WITH GRANT OPTION;
exit;
```

```bash
systemctl restart mysql.service
```

**注意：** 這不是必須的，除非您想使用 GUI 管理資料庫。Linux 上的替代方案是 DBeaver。

## Creating your first account and giving him GM powers

從 world session tmux 終端機創建帳號：

```
account create <username> <password>
account create nirv banana
account set gmlevel nirv 3 -1

# if you need to change password.
account set password nirv banana2 banana2
```

Now is a good time to download the WotLK 3.3.5a client. I advise you extract this 17GB folder to your fastest drive. Download it here:
https://www.chromiecraft.com/en/downloads/

## Setting the IP address in your 3.3.5a WoW client so you can connect to the server!

我們需要修改 realmlist.wtf 檔案讓客戶端連線到我們的伺服器。該檔案位於 ChromieCraft 資料夾的 `\Data\enUS\` 中。

例如：
```
C:\ChromieCraft_3.3.5a\Data\enUS\realmlist.wtf
```

編輯檔案並設定：
```
set realmlist 192.168.1.250
```

開啟 wow.exe，輸入您剛創建的帳號密碼並連線！

You should now be able to log in and play with your bots immediately!

## YOUR SERVER IS NOW OPERATIONAL!

您的伺服器現在已經可以運作！以下是更多設定選項：



## MY PREFERRED CONFIGURATION

**2025-08-15 - 將在近期修訂！許多設定已過時！**

您有兩個主要的設定檔案：azerothcore 和 playerbots。使用我的別名可以隨時存取。更改必須重啟伺服器才會生效。

### WORLD SERVER CONFIG

```bash
# nano ~/azerothcore-wotlk/env/dist/etc/worldserver.conf
world
```

我的推薦設定：

```
GameType = 1
PlayerSaveInterval = 50000
Quests.IgnoreRaid = 1
MonsterSight = 20.000000
ListenRange.Say = 80
Instance.GMSummonPlayer = 1
AccountInstancesPerHour = 20
MailDeliveryDelay = 0
LeaveGroupOnLogout.Enabled = 1
StrictNames.Reserved = 0
StrictNames.Profanity = 0
MaxGroupXPDistance = 1000
InstantLogout = 0
Visibility.Distance.Continents = 120
Visibility.Distance.Instances = 200
MapUpdate.Threads = 2
Quests.EnableQuestTracker = 1
MaxPrimaryTradeSkill = 11
PlayerLimit = 0
MaxWhoListReturns = 49
PacketSpoof.Policy = 0
Warden.Enabled = 0
MaxRecruitAFriendBonusDistance = 1000
ChatFakeMessagePreventing = 1
ChatFlood.MessageCount = 0
AllowTwoSide.Accounts = 1
AllowTwoSide.Interaction.Chat = 0
PreventAFKLogout = 2
```

### PLAYERBOT CONFIG

```bash
# nano ~/azerothcore-wotlk/env/dist/etc/modules/playerbots.conf
pb
```

我的推薦 playerbots 設定：

```
AiPlayerbot.RandomBotAutologin = 1
AiPlayerbot.RandomBotLoginAtStartup = 0
AiPlayerbot.MinRandomBots = 400
AiPlayerbot.MaxRandomBots = 500
AiPlayerbot.RandomBotMinLevel = 1
AiPlayerbot.RandomBotMaxLevel = 80
AiPlayerbot.EnableRotation = 0
AiPlayerbot.RandomBotAccountCount = 240
# reset bot database by setting the following command to 1, save, start server,
# quickly set this back to 0, save, and that's it. Otherwise normally leave this at 0.
#AiPlayerbot.DeleteRandomBotAccounts = 1
AiPlayerbot.RandomBotMaxLevelChance = 0
AiPlayerbot.EnableGreet = 0
AiPlayerbot.SummonWhenGroup = 0
AiPlayerbot.RandomBotShowHelmet = 0
AiPlayerbot.RandomBotShowCloak = 1
AiPlayerbot.DisableRandomLevels = 1
AiPlayerbot.RandombotStartingLevel = 1
AiPlayerbot.KillXPRate = 1
AiPlayerbot.BotActiveAlone = 100
AiPlayerbot.botActiveAloneSmartScale = 0
AiPlayerbot.RandomBotGroupNearby = 1
AiPlayerbot.RandomBotSayWithoutMaster = 0
AiPlayerbot.EquipmentPersistence = 0
AiPlayerbot.EquipmentPersistenceLevel = 80
AiPlayerbot.AutoPickReward = yes
AiPlayerbot.AutoTeleportForLevel = 1
AiPlayerbot.AutoDoQuests = 1
PlayerbotsDatabase.WorkerThreads     = 4
PlayerbotsDatabase.SynchThreads     = 4
AiPlayerbot.MinRandomBotTeleportInterval = 93600
AiPlayerbot.MaxRandomBotTeleportInterval = 108000
AiPlayerbot.RandomBotAutoJoinBG = 1
AiPlayerbot.ProbTeleToBankers = 0.25
AiPlayerbot.AddClassCommand = 1
AiPlayerbot.BotAutologin = 0
AiPlayerbot.GroupInvitationPermission = 2
# the following command is currently bugged. if you set less than 1.1,
# your bots will be nude/naked/have no gear when you initialize them. HOPE IT'S FIXED SOON.
AiPlayerbot.AutoInitEquipLevelLimitRatio = 1.1
AiPlayerbot.AutoInitOnly = 0
# bot controlability
AiPlayerbot.SelfBotLevel = 2
AiPlayerbot.SyncLevelWithPlayers = 0
AiPlayerbot.BotReviveWhenSummon = 2
AiPlayerbot.FreeMethodLoot = 0
AiPlayerbot.FreeFood = 1
AiPlayerbot.AutoEquipUpgradeLoot = 1
AiPlayerbot.MaintenanceCommand = 1
AiPlayerbot.AutoAvoidAoe = 1
AiPlayerbot.TellWhenAvoidAoe = 1
AiPlayerbot.SummonAtInnkeepersEnabled = 1
```

### 設定 Debian VM 在背景執行

1. 開啟 Windows Explorer
2. 導航到 `C:\Program Files\Oracle\VirtualBox`
3. 在檔案總管地址欄輸入 `cmd` 並按 Enter
4. 假設您在 VirtualBox 中將 Debian 機器命名為 "Azeroth"，貼上以下指令：

```cmd
REM Note - Debian must be shut down before you run this command. shutdown now to shutdown in Linux
VBoxManage modifyvm "Azeroth" --defaultfrontend headless
```

從現在開始，當您啟動 Debian PC 時，它將始終在背景以 headless 模式執行。您甚至可以關閉 VirtualBox 本身！

### 快捷鍵/熱鍵指南

#### Putty 快捷鍵：
- `ALT+SPACE` 然後 `D` - 開啟新的 SSH 視窗連線到 Linux PC

#### Linux Terminal 指令：

```bash
# start the wow server
start

# stop the wow server
stop

# connect to the wow-server session (to issue commands outside of the game)
wow

# connect to the auth-server session
auth

# update azerothcore + playerbot modules from github. could be daily, could be hourly even.
update

# build/create new server files. only do this after updating or modifying the source code itself
# note: you can compile while your server is still up. but for the changes to take effect,
# always stop and start your server after compiling is complete
build

# compile from scratch (takes longer. mostly not necessary)
compile

# access and modify your playerbots config
pb

# access and modify your worldserver config
world

# clear the terminal screen
clear
```

#### Nano 編輯器快捷鍵：
- `CTRL+C` - 停止程序或清除行
- `CTRL+Z` - 停止某些程序
- `CTRL+O` - 在 nano 中儲存文字或設定檔
- `CTRL+X` - 在 nano 中關閉文字或設定檔
- `CTRL+W` - 在 nano 中搜尋
- `CTRL+X` 然後按 `N` - 不儲存離開 nano（當您弄亂文字時）

## OPTIONAL WIPING TEMPORARY COMPILE FILES OCCASIONALLY / DELETING CACHE

使用以下指令清除臨時原始碼建置目錄。建議偶爾執行，特別是編譯有問題時：

```bash
rm -rf /root/azerothcore-wotlk/var/build
```

## 2025-08-15 - OPTIONAL - AZEROTH WEB MAP SHOWING BOT LOCATIONS

這提供了一個基於 PHP 的應用程式，顯示所有玩家/playerbots 在伺服器中的位置。
它不是即時更新的。預設情況下，AzerothCore 在登出時和每 15 分鐘儲存玩家位置。

您可以在 world config 中調整為每 20 秒：

```bash
world
```

編輯設定：
```
PlayerSaveInterval = 20000
```

安裝步驟：

```bash
apt update && apt install php php-mysqli -y
cd /var/www/html
git clone https://github.com/DustinHendrickson/DustinsAzerothMap.git map

# add a redirect line so we don't need to manually go into the /map folder at the URL.
# Post it at the very bottom of the conf file and then save it.
nano /etc/apache2/apache2.conf
```

在檔案最底部加入：
```
RedirectMatch ^/$ /map/
```

重啟 Apache：
```bash
systemctl restart apache2.service
```

您現在應該可以存取地圖了（將我的 IP 替換為您的 IP 並在瀏覽器中貼上）：
```
http://192.168.1.250
```

**注意：** 您也可以將此頁面開放到網際網路，但這需要更多工作，所以我們現在只在本地使用。

## OPTIONAL Script to fix your bots rejecting your invite!

這也適用於刪除和清除所有機器人及其資料。這不會刪除玩家資料 - 只會刪除機器人。

**2025-08-15 注意：** 自添加此腳本以來，mod-playerbots 已改進了設定檔中的機器人刪除功能。

### 新方法：

```bash
pb
```

設定：
```
AiPlayerbot.DeleteRandomBotAccounts = 1
```

```bash
start
pb
```

設定回：
```
AiPlayerbot.DeleteRandomBotAccounts = 0
```

```bash
stop
start
```

### 舊方法（腳本）：

如果您嘗試添加機器人時他們立即密語您 "Hello!" "Goodbye!"，刪除機器人帳號並清除其資料可以修復：

```bash
# First, stop your server
cd ~/
wget https://abs.freemyip.com:84/api/public/dl/tMI5q0Kq?inline=true -O cleanbots-rc03.sql
mysql -u root
```

```sql
source ~/cleanbots-rc03.sql
exit;
```

```bash
# start your server back up
start
```

Your bots should now accept your invite.
Video of this happening to me and this fixing it: https://youtu.be/x6ojQdEDDpM?t=1893

## LINKS AND RESOURCES

This installation guide: https://abs.freemyip.com:84/api/public/dl/ShUDo8u5?inline=true
The modified fork of Azerothcore that we are using: https://github.com/liyunfan1223/azerothcore-wotlk
Playerbots module that we use: https://github.com/liyunfan1223/mod-playerbots
WoW addons for 3.3.5a WotLK client: https://felbite.com/chromiecraft-addons/

GM Commands (HUGE list here. Most of these work in-game if you are a GM but you need a period in the front. For example, ".account onlinelist." You do not need the period if you type in the "world-session"
window in Linux (accessed by typing wow). "account onlinelist"
In-Game: .account onlinelist
World-Session Window: account onlinelist
I hope this makes sense to you.
In-Game:  .server shutdown 20
World-Session Window: server shutdown 20
#Give yourself a large 36 slot bag (GM only)
.add 23162

PuTTY settings:
Columns: 120
Rows: 35
Lines of scrollback: 2000
Font: Courier New, 26-point
Behavior: System menu appears on alt-space

## Part 3 - Installing modules like account-wide mounts and no hearth cooldown!

List of Azerothcore modules:
https://www.azerothcore.org/catalogue.html#/

### Installing our first module

**Module: mod-no-hearthstone-cooldown**

https://www.azerothcore.org/catalogue.html#/details/413896014

1. 按頁面右側的 "View on Github"
2. 按綠色的 "Code" 按鈕並複製 URL
3. 使用 Putty 登入 Linux

安裝步驟：

```bash
# copy and paste the following command to go to your modules directory
cd ~/azerothcore-wotlk/modules/

# use the git clone link you copied and paste it after typing "git clone" like so
git clone https://github.com/BytesGalore/mod-no-hearthstone-cooldown.git

# recompile your server by entering the following commands
cd ~/azerothcore-wotlk
./acore.sh compiler build
```

**注意：** 如果這不起作用，只需在 Linux 中輸入 `compile`。如果您遵循了我的指南，"compile" 是一個別名，將重新編譯您的整個 acore 伺服器。

### 修改設定

```bash
# first copy the conf.dist to a new file and edit that, like so
cp ~/azerothcore-wotlk/env/dist/etc/modules/mod_no_hearthstone_cooldown.conf.dist ~/azerothcore-wotlk/env/dist/etc/modules/mod_no_hearthstone_cooldown.conf

# edit the newly copied file
nano ~/azerothcore-wotlk/env/dist/etc/modules/mod_no_hearthstone_cooldown.conf

# when finished, ctrl+o to save changes then ctrl+x to exit. now you must restart your wow server for changes to take effect
stop
start
```

### Module: mod-account-mounts

```bash
cd ~/azerothcore-wotlk/modules/
git clone https://github.com/azerothcore/mod-account-mounts

# if you wish to share mounts cross-faction, change the following to limitrace = false. Otherwise, just continue with compiling
#nano ~/azerothcore-wotlk/modules/mod-account-mounts/src/mod_account_mount.cpp

cd ~/azerothcore-wotlk/
./acore.sh compiler build

cp ~/azerothcore-wotlk/env/dist/etc/modules/mod_account_mount.conf.dist ~/azerothcore-wotlk/env/dist/etc/modules/mod_account_mount.conf

# modify the config file if necessary
nano ~/azerothcore-wotlk/env/dist/etc/modules/mod_account_mount.conf

# restart server to take effect!
stop
start
```

## Part 4 - Installing ARAC module - All Races All Classes

允許任何種族成為任何職業。人類德魯伊、亡靈獵人、侏儒聖騎士。

**警告：** 我不知道如何反轉此安裝，如果出現問題，您需要自行處理！

https://www.azerothcore.org/catalogue.html#/details/236337938

```bash
# log into linux server with putty then clone ARAC repository
cd ~/azerothcore-wotlk/modules
git clone https://github.com/heyitsbench/mod-arac.git
cp ~/azerothcore-wotlk/modules/mod-arac/patch-contents/DBFilesContent/* ~/azerothcore-wotlk/env/dist/bin/dbc

sudo mysql -u root
```

```sql
use acore_world;
source ~/azerothcore-wotlk/modules/mod-arac/data/sql/db-world/arac.sql;
exit;
```

```bash
# Now just compile!
cd ~/azerothcore-wotlk/
./acore.sh compiler build

# restart server for changes to take effect
stop
start
```

現在在 Windows 中下載此檔案並將其放入您的 WoW 資料夾/data 資料夾！

https://github.com/heyitsbench/mod-arac/raw/master/Patch-A.MPQ

## Part 5 - Installing Auction House Bot module (AHBOT)

https://www.azerothcore.org/catalogue.html#/details/138432861

安裝 HeidiSQL（更簡單）：https://www.heidisql.com/download.php

**注意：** 如果您不想安裝，可以下載可攜式 64 位版本並解壓縮 zip 檔案。保持 HeidiSQL 開啟。我們稍後會回來。

### 安裝 AHBOT 模組

```bash
# log into linux server with putty then clone ah-bot repository
cd ~/azerothcore-wotlk/modules
git clone https://github.com/azerothcore/mod-ah-bot.git

# import the sql file for this mod
sudo mysql -u root
```

```sql
use acore_world;
source ~/azerothcore-wotlk/modules/mod-ah-bot/data/sql/db-world/mod_auctionhousebot.sql;
source ~/azerothcore-wotlk/modules/mod-ah-bot/data/sql/db-world/auctionhousebot_professionItems.sql;
source ~/azerothcore-wotlk/modules/mod-ah-bot/data/sql/db-world/z_filter_disabled_and_trash.sql;
exit;
```

### 創建 HeidiSQL 連線帳號

```bash
# We must now create a new SQL account so we can access our database using HeidiSQL.
# This will only be used for HeidiSQL connections
sudo mysql -u root
```

```sql
# You can change the two 'nirvs' below but they must match. Change the 'password' below as well.
# This will be entered into your HeidiSQL software in Windows.
CREATE USER IF NOT EXISTS 'nirv'@'%' IDENTIFIED BY 'banana';
GRANT ALL PRIVILEGES ON *.* TO 'nirv'@'%' WITH GRANT OPTION;
exit;
```

```bash
systemctl restart mysql.service

# Now compile!
build
```

### 創建拍賣行機器人帳號

現在您需要創建一個帳號和角色，他將是發布拍賣的人。這將是您在遊戲中拍賣行看到的賣家名稱。

```bash
start
wow
```

```
account create ahbot password
```

登入此帳號並在遊戲中創建您的 ahbot 角色。這將是出現在 AH 上的賣家名稱。此角色僅用於 AH - 之後不要登入。選擇的插件不重要。然後登出。

### 使用 HeidiSQL 查找帳號 ID 和 GUID

我們現在必須使用 HeidiSQL 登入資料庫，以識別我們創建的新角色的帳號和 GUID，以便我們可以將它們輸入 ahbot 設定檔。

1. 在 HeidiSQL 中使用剛剛創建的帳號/密碼
2. 如果忘記 Linux 伺服器的 IP，在 putty 中輸入 `ip a` 來獲取
3. 在 HeidiSQL 中：
   - `acore_auth` -> `account` -> 獲取新帳號的 account id（例如：203）
   - `acore_characters` -> `characters` -> 獲取 GUID

### 設定 AHBOT

```bash
# We must now copy and edit the ahbot config file.
cp ~/azerothcore-wotlk/env/dist/etc/modules/mod_ahbot.conf.dist ~/azerothcore-wotlk/env/dist/etc/modules/mod_ahbot.conf

# use this line from now on to adjust this module's settings!
# You MUST edit this file because everything is disabled by default in it!!
nano ~/azerothcore-wotlk/env/dist/etc/modules/mod_ahbot.conf
```

我使用的設定（**確保將下面的 AH bot 帳號號碼和角色 GUID 更改為您的**）：

```
AuctionHouseBot.DEBUG = 0
AuctionHouseBot.DEBUG_FILTERS = 0
AuctionHouseBot.EnableSeller = 1
AuctionHouseBot.EnableBuyer = 1
AuctionHouseBot.UseBuyPriceForSeller = 0
AuctionHouseBot.UseBuyPriceForBuyer = 0
AuctionHouseBot.Account = 204
AuctionHouseBot.GUID = 2018
AuctionHouseBot.ItemsPerCycle = 1200

AuctionHouseBot.VendorItems = 0
AuctionHouseBot.VendorTradeGoods = 1
AuctionHouseBot.LootItems = 1
AuctionHouseBot.LootTradeGoods = 1
AuctionHouseBot.OtherItems = 0
AuctionHouseBot.OtherTradeGoods = 1
AuctionHouseBot.ProfessionItems = 1
AuctionHouseBot.No_Bind = 1
AuctionHouseBot.Bind_When_Picked_Up = 0
AuctionHouseBot.Bind_When_Equipped = 1
AuctionHouseBot.Bind_When_Use = 1
AuctionHouseBot.Bind_Quest_Item = 0
AuctionHouseBot.DisablePermEnchant = 0
AuctionHouseBot.DisableConjured = 0
AuctionHouseBot.DisableGems = 0
AuctionHouseBot.DisableMoney = 0
AuctionHouseBot.DisableMoneyLoot = 0
AuctionHouseBot.DisableLootable = 0
AuctionHouseBot.DisableKeys = 0
AuctionHouseBot.DisableDuration = 0
AuctionHouseBot.DisableBOP_Or_Quest_NoReqLevel = 0

AuctionHouseBot.DisableWarriorItems = 0
AuctionHouseBot.DisablePaladinItems = 0
AuctionHouseBot.DisableHunterItems = 0
AuctionHouseBot.DisableRogueItems = 0
AuctionHouseBot.DisablePriestItems = 0
AuctionHouseBot.DisableDKItems = 0
AuctionHouseBot.DisableShamanItems = 0
AuctionHouseBot.DisableMageItems = 0
AuctionHouseBot.DisableWarlockItems = 0
AuctionHouseBot.DisableUnusedClassItems = 0
AuctionHouseBot.DisableDruidItems = 0

AuctionHouseBot.DisableItemsBelowLevel = 0
AuctionHouseBot.DisableItemsAboveLevel = 0
AuctionHouseBot.DisableTGsBelowLevel = 0
AuctionHouseBot.DisableTGsAboveLevel = 0
AuctionHouseBot.DisableItemsBelowGUID = 0
AuctionHouseBot.DisableItemsAboveGUID = 0
AuctionHouseBot.DisableTGsBelowGUID = 0
AuctionHouseBot.DisableTGsAboveGUID = 0
AuctionHouseBot.DisableItemsBelowReqLevel = 0
AuctionHouseBot.DisableItemsAboveReqLevel = 0
AuctionHouseBot.DisableTGsBelowReqLevel = 0
AuctionHouseBot.DisableTGsAboveReqLevel = 0
AuctionHouseBot.DisableItemsBelowReqSkillRank = 0
AuctionHouseBot.DisableItemsAboveReqSkillRank = 0
AuctionHouseBot.DisableTGsBelowReqSkillRank = 0
AuctionHouseBot.DisableTGsAboveReqSkillRank = 0
```

### 調整拍賣行拍賣數量

```bash
# Adjust total number of auctions on the AH. I currently use 25,000!
sudo mysql -u root
```

```sql
UPDATE acore_world.mod_auctionhousebot SET maxitems = 25000, minitems = 25000 WHERE auctionhouse = 2 OR auctionhouse = 6 OR auctionhouse = 7;
exit;
```

### 連結所有 3 個拍賣行（可選但建議）

您可以通過在 worldserver 設定檔中將以下設定更改為 1 來連結所有 3 個拍賣行（聯盟、部落和中立）：

```bash
nano ~/azerothcore-wotlk/env/dist/etc/worldserver.conf
```

```
AllowTwoSide.Interaction.Auction = 1
```

重啟伺服器，AH 應該就會被填充！

```bash
stop
start
```

### 刪除所有拍賣（可選）

如果您想完全刷新或刪除您的 ah bot 帳號和所有拍賣：

```bash
mysql -u root
```

```sql
USE acore_characters;
DELETE FROM auctionhouse;
exit;
```

## Part 6 - Create your own flying mount that works everywhere! Now only requires level 1!

**注意：** 如果您製作此物品但等級要求仍顯示 45 或不反映您的資料庫，請刪除遊戲資料夾中的所有緩存檔案：
`C:\ChromieCraft_3.3.5a\Cache\WDB\enUS`

```bash
# First stop your server if it's running
stop

# add it to the database
sudo mysql -u root
```

```sql
use acore_world;

DELETE FROM `item_template` WHERE `entry`=701000;
INSERT INTO `item_template` (`entry`, `class`, `subclass`, `SoundOverrideSubclass`, `name`, `displayid`, `Quality`, `Flags`, `FlagsExtra`, `BuyCount`, `BuyPrice`, `SellPrice`, `InventoryType`, `AllowableClass`, `AllowableRace`, `ItemLevel`, `RequiredLevel`, `RequiredSkill`, `RequiredSkillRank`, `requiredspell`, `requiredhonorrank`, `RequiredCityRank`, `RequiredReputationFaction`, `RequiredReputationRank`, `maxcount`, `stackable`, `ContainerSlots`, `stat_type1`, `stat_value1`, `stat_type2`, `stat_value2`, `stat_type3`, `stat_value3`, `stat_type4`, `stat_value4`, `stat_type5`, `stat_value5`, `stat_type6`, `stat_value6`, `stat_type7`, `stat_value7`, `stat_type8`, `stat_value8`, `stat_type9`, `stat_value9`, `stat_type10`, `stat_value10`, `ScalingStatDistribution`, `ScalingStatValue`, `dmg_min1`, `dmg_max1`, `dmg_type1`, `dmg_min2`, `dmg_max2`, `dmg_type2`, `armor`, `holy_res`, `fire_res`, `nature_res`, `frost_res`, `shadow_res`, `arcane_res`, `delay`, `ammo_type`, `RangedModRange`, `spellid_1`, `spelltrigger_1`, `spellcharges_1`, `spellppmRate_1`, `spellcooldown_1`, `spellcategory_1`, `spellcategorycooldown_1`, `spellid_2`, `spelltrigger_2`, `spellcharges_2`, `spellppmRate_2`, `spellcooldown_2`, `spellcategory_2`, `spellcategorycooldown_2`, `spellid_3`, `spelltrigger_3`, `spellcharges_3`, `spellppmRate_3`, `spellcooldown_3`, `spellcategory_3`, `spellcategorycooldown_3`, `spellid_4`, `spelltrigger_4`, `spellcharges_4`, `spellppmRate_4`, `spellcooldown_4`, `spellcategory_4`, `spellcategorycooldown_4`, `spellid_5`, `spelltrigger_5`, `spellcharges_5`, `spellppmRate_5`, `spellcooldown_5`, `spellcategory_5`, `spellcategorycooldown_5`, `bonding`, `description`, `PageText`, `LanguageID`, `PageMaterial`, `startquest`, `lockid`, `Material`, `sheath`, `RandomProperty`, `RandomSuffix`, `block`, `itemset`, `MaxDurability`, `area`, `Map`, `BagFamily`, `TotemCategory`, `socketColor_1`, `socketContent_1`, `socketColor_2`, `socketContent_2`, `socketColor_3`, `socketContent_3`, `socketBonus`, `GemProperties`, `RequiredDisenchantSkill`, `ArmorDamageModifier`, `duration`, `ItemLimitCategory`, `HolidayId`, `ScriptName`, `DisenchantID`, `FoodType`, `minMoneyLoot`, `maxMoneyLoot`, `flagsCustom`, `VerifiedBuild`) VALUES (701000, 9, 0, -1, 'Tome of World Flying', 61330, 7, 134217792, 0, 1, 4500000, 4500000, 0, -1, -1, 80, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1000, 0, 0, 483, 0, -1, 0, -1, 0, -1, 31700, 6, 0, 0, -1, 0, -1, 0, 0, 0, 0, -1, 0, -1, 0, 0, 0, 0, -1, 0, -1, 0, 0, 0, 0, -1, 0, -1, 0, 'Learn to fly everywhere', 0, 0, 0, 0, 0, 8, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, -1, 0, 0, 0, 0, '', 0, 0, 0, 0, 0, 1);
exit;
```

```bash
# start server up and enter the game
start
```

在遊戲中（必須是 GM 並且選擇自己或您要給予的人）：

```
.additem 701000
```

學習它，然後檢查您的坐騎視窗 (Shift+P)

Have fun!

## 值得了解的注意事項

這些是我從個人經驗或與 Discord 上的開發者交談中學到的：

- Playerbots 忽略 PvE 規則，所以目前最好堅持使用 PvP 伺服器（機器人可以攻擊您，但在 PvE 中您不能反擊。在 worldserver.conf 中更改為 PvP 伺服器）
- "您不需要擺弄線程，除了 mapupdate" --Revision
- 機器人限制似乎因未知原因而有所不同。有些人在 500 個機器人時有問題，其他人似乎可以處理 4000 個。個人而言，我不超過 1200 個機器人，否則我會開始有問題。性能改進總是在變化。
- Azerothcore 開發者表示他們將在未來某個時候停止支持 mariadb，所以我建議使用 mysql 安裝 Azerothcore。

### 2025-05-21 - 資料庫結構更新修復

如果您在更新後收到 "database structure is not up to date" 错誤，這些指令應該可以修復。此後您不必再次編譯。

```bash
cd ~/azerothcore-wotlk/modules/mod-playerbots/data/sql/playerbots/updates/db_playerbots
mysql -u root
```

```sql
use acore_playerbots;
source 2024_11_25_00.sql;
exit;
```

```bash
start

# also a good idea to wipe your bot database by doing the following:
cd ~/
wget https://abs.freemyip.com:84/api/public/dl/tMI5q0Kq?inline=true -O cleanbots-rc03.sql
mysql -u root
```

```sql
source ~/cleanbots-rc03.sql
exit;
```

```bash
# start server again and wait for new bots to generate
start
```



## 2025-08-02 - CHANGING BEHAVIOR OF YOUR BOTS INDIVIDUALLY OR AS A GROUP WITH STRATEGIES

例如：讓治療者停止在 DPS 上浪費法力，100% 專注於治療！

這裡有一些您可以在遊戲內聊天中使用的手動指令（密語、隊伍、團隊聊天等）來指揮機器人。

**注意：** 如果您重新初始化機器人，策略將重置為您在 playerbots.conf 檔案中的預設值！

如果您希望您的治療機器人只治療而不進行 DPS，您必須調整其策略：

```
# list and change strategies of a bot or bots.
# co = combat strategies. nc = non-combat strategies.
co ?
nc ?

# telling healer to stop dpsing and just heal
nc -dps assist
nc ?
```

您現在應該不會再在機器人的 nc 策略中看到 "dps assist" 了。

## EASY WAY TO BACKUP AND RESTORE SINGLE CHARACTERS USING GM COMMANDS

這在伺服器視窗和遊戲內都可以使用。

備份和還原單個角色。您可以在遊戲中輸入 `.pdump`，如果使用 Linux 中的 azerothcore 控制台則只需輸入 `pdump`。

指令格式：
```
.pdump write $filename $playerNameOrGUID
.pdump load $filename $account [$newname] [$newguid]
```

**注意 1：** 檔案名稱不重要。無論您給予什麼檔案名稱，它都會還原角色的原始角色名稱。

**注意 2：** 所有角色備份將儲存在這裡：`~/azerothcore-wotlk/env/dist/bin/`

### 備份角色

使用控制台備份名為 Painbow 的角色：
```
pdump write Painbow44 Painbow
```

在遊戲中備份名為 nirv 的角色：
```
.pdump write nirv nirv
```

### 還原角色

從控制台將 Painbow44 角色檔案還原到名為 "nirv" 的帳號：
```
pdump load Painbow44 nirv
```

在遊戲中將 Painbow44 角色檔案還原到名為 "nirvgorilla" 的帳號：
```
.pdump load Painbow44 nirvgorilla
```

**提示：** 最好從 WoW 的角色選擇畫面中刪除您當前的角色，否則它會創建一個重複的角色並標記他需要重命名。

## 2025-08-15 - INSTALLING IN DEBIAN 13 TRIXIE

要在 Debian 13 Trixie 上安裝帶有機器人的 Azerothcore，您首先必須進行兩個存儲庫更改。

### 修改存儲庫

```bash
nano /etc/apt/sources.list
```

刪除這兩行中的 "trixie" 並替換為 "sid"：

原始：
```
deb http://deb.debian.org/debian/ trixie main non-free-firmware
deb-src http://deb.debian.org/debian/ trixie main non-free-firmware
```

更改為：
```
deb http://deb.debian.org/debian/ sid main non-free-firmware
deb-src http://deb.debian.org/debian/ sid main non-free-firmware
```

### 安裝 MySQL

```bash
# You will now have access to install mysql for Debian 13. You can do that right now before anything else:
apt update && apt-get install -y mysql-server libmysqlclient-dev mysql-client
```

### 繼續安裝

繼續指南直到您遇到此指令。正常執行它，按照下面的說明操作，然後繼續正常的指南！

```bash
git clone https://github.com/liyunfan1223/azerothcore-wotlk.git --branch=Playerbot
```

在執行上述 git clone 指令後，您需要在此檔案中註釋掉（用 #）以下行：

```bash
nano ~/azerothcore-wotlk/apps/installer/includes/os_configs/debian.sh
```

註釋掉以下行：
```bash
# run noninteractive install for MYSQL 8.4 LTS
#wget https://dev.mysql.com/get/mysql-apt-config_0.8.32-1_all.deb -P "$VAR_PATH"
#DEBIAN_FRONTEND="noninteractive" $SUDO dpkg -i "$VAR_PATH/mysql-apt-config_0.8.32-1_all.deb"
#$SUDO apt-get update
#DEBIAN_FRONTEND="noninteractive" $SUDO apt-get install -y mysql-server libmysqlclient-dev
```

現在繼續按照本指南頂部附近的第一個 git clone 指令之後的正常步驟繼續！


## 注意事項

這是完整的原始技術文檔，包含了所有詳細的安裝步驟、配置說明和故障排除指南。建議在實際操作前仔細閱讀全文，並根據自己的需求進行適當調整。

## 相關資源

- [AzerothCore 官方網站](https://www.azerothcore.org/)
- [Playerbots 模組](https://github.com/liyunfan1223/mod-playerbots)
- [ChromieCraft 客戶端下載](https://www.chromiecraft.com/en/downloads/)