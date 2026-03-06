+++
title = 'Neovim 系列（五）：安裝 Neovim 與你的第一個 init.lua'
date = 2026-02-27T10:00:00+08:00
draft = false
tags = ['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列', 'Lua']
categories = ['技術筆記']
author = 'Jack'
description = '從零開始安裝 Neovim，認識 Lua 配置系統，設定基本選項、Leader Key 和常用快捷鍵映射，打造你的第一個 init.lua 配置檔'
toc = true
weight = 5
+++

> 這是 **Neovim 從零開始**系列的第五篇。整個系列共 12 篇文章，將帶你從完全不懂 Vim，到能用 Neovim 打造一個完整的現代化開發環境。

## 從練習到實戰

前四篇我們學完了 Vim 操作的核心基礎，你可能已經在 VSCode + Vim extension 中練習了一陣子。現在，是時候正式安裝 Neovim，開始打造屬於自己的編輯器了。

這篇會帶你從安裝開始，到寫出一個實用的 `init.lua` 配置檔。

## 安裝 Neovim

### macOS（推薦用 Homebrew）

```bash
brew install neovim
```

安裝完成後確認版本：

```bash
nvim --version
```

建議使用 **0.10+** 以上的版本，以確保內建 LSP 和 Treesitter 等功能正常運作。

### 其他平台

```bash
# Ubuntu / Debian
sudo apt install neovim

# Arch Linux
sudo pacman -S neovim

# Windows（用 winget）
winget install Neovim.Neovim
```

### 第一次啟動

```bash
nvim
```

你會看到一個歡迎畫面。按 `:q` 離開。恭喜，你已經成功啟動了 Neovim！

目前它看起來很陽春——沒有行號、沒有語法高亮、沒有任何自訂設定。接下來我們會一步步改善。

## 配置檔的位置

Neovim 的配置檔放在：

| 平台 | 路徑 |
|------|------|
| macOS / Linux | `~/.config/nvim/init.lua` |
| Windows | `~/AppData/Local/nvim/init.lua` |

先建立目錄和檔案：

```bash
mkdir -p ~/.config/nvim
touch ~/.config/nvim/init.lua
```

Neovim 啟動時會自動載入這個檔案。之後所有的設定都從這裡開始。

## 為什麼用 Lua？

Neovim 支援兩種配置語言：

- **Vimscript**：Vim 傳統的腳本語言，歷史悠久但語法不太直覺
- **Lua**：Neovim 原生整合的現代語言，速度快、語法清晰

本系列完全使用 **Lua**。原因：

1. 語法對程式設計師更友善
2. 執行速度更快
3. 現代 Neovim plugin 幾乎都用 Lua
4. 更好的錯誤提示

### Lua 快速入門

你不需要先學 Lua——邊寫配置邊學就好。以下是最常用的語法：

```lua
-- 這是註解

-- 變數
local name = "Jack"
local count = 42
local enabled = true

-- 表（Table）—— Lua 唯一的資料結構，兼具 array 和 object
local list = { "a", "b", "c" }
local config = { width = 80, height = 24 }

-- 函數
local function greet(name)
  return "Hello, " .. name  -- .. 是字串串接
end

-- 條件判斷
if count > 10 then
  print("large")
elseif count > 0 then
  print("small")
else
  print("zero")
end

-- 迴圈
for i = 1, 10 do
  print(i)
end

for _, item in ipairs(list) do
  print(item)
end
```

## 基本選項設定

Neovim 的選項透過 `vim.opt` 來設定。打開 `~/.config/nvim/init.lua`，開始加入設定：

### 行號顯示

```lua
vim.opt.number = true         -- 顯示行號
vim.opt.relativenumber = true -- 顯示相對行號
```

相對行號非常重要——它讓你可以快速知道目標行距離幾行，直接用 `5j` 或 `12k` 跳過去。

```
  3  function hello()
  2    local x = 1
  1    local y = 2
  7  ← 游標在這行（顯示絕對行號）
  1    return x + y
  2  end
  3
```

### 縮排設定

```lua
vim.opt.tabstop = 4        -- Tab 顯示寬度
vim.opt.shiftwidth = 4     -- 自動縮排寬度
vim.opt.expandtab = true   -- 用空格取代 Tab
vim.opt.smartindent = true -- 智能縮排
```

### 搜尋設定

```lua
vim.opt.ignorecase = true  -- 搜尋忽略大小寫
vim.opt.smartcase = true   -- 如果搜尋包含大寫就區分大小寫
vim.opt.hlsearch = true    -- 高亮搜尋結果
vim.opt.incsearch = true   -- 即時顯示搜尋匹配
```

`smartcase` 的巧妙之處：搜尋 `/hello` 不分大小寫，但搜尋 `/Hello` 就會精確匹配大寫。

### 外觀設定

```lua
vim.opt.termguicolors = true  -- 啟用 24-bit 色彩
vim.opt.cursorline = true     -- 高亮目前所在行
vim.opt.signcolumn = "yes"    -- 永遠顯示 sign column（避免畫面跳動）
vim.opt.scrolloff = 8         -- 游標上下保留 8 行可視範圍
vim.opt.colorcolumn = "80"    -- 在第 80 列顯示參考線
```

`scrolloff = 8` 讓你在滾動時永遠能看到游標上下各 8 行的內容，不會出現「游標在螢幕最底部」的情況。

### 系統整合

```lua
vim.opt.clipboard = "unnamedplus"  -- 和系統剪貼簿同步
vim.opt.mouse = "a"                -- 啟用滑鼠（過渡期有用）
vim.opt.undofile = true            -- 持久化 undo 歷史（關閉檔案後還能 undo）
```

`clipboard = "unnamedplus"` 讓你在 Neovim 中 `y` 複製的內容可以直接在其他程式中 `Cmd+V` 貼上，反之亦然。

### 分割視窗

```lua
vim.opt.splitbelow = true   -- 水平分割時新視窗在下方
vim.opt.splitright = true   -- 垂直分割時新視窗在右方
```

### 其他實用設定

```lua
vim.opt.wrap = false           -- 不自動換行
vim.opt.swapfile = false       -- 不建立 swap 檔案
vim.opt.backup = false         -- 不建立備份檔案
vim.opt.updatetime = 250       -- 減少延遲（預設 4000ms）
vim.opt.timeoutlen = 300       -- 快捷鍵組合的等待時間
vim.opt.completeopt = { "menuone", "noselect" }  -- 補全選單設定
```

## Leader Key

Leader Key 是 Vim 中自訂快捷鍵的核心概念。它是一個「前綴鍵」——按下它之後再按其他鍵，就觸發自訂功能。

### 設定 Leader Key

```lua
vim.g.mapleader = " "       -- 設定 leader 為空白鍵
vim.g.maplocalleader = " "  -- 設定 local leader 也為空白鍵
```

**為什麼用空白鍵？**

- 位置在鍵盤正中央，兩手拇指都能按到
- 在 Normal mode 下空白鍵的預設功能（向右移動）和 `l` 重複，不會損失功能
- 幾乎所有現代 Neovim 配置都用空白鍵

設定後，`<leader>` 就代表空白鍵。例如 `<leader>w` 就是「空白鍵 + w」。

> 重要：Leader key 的設定必須在所有 keymap 設定之前。

## 快捷鍵映射（Keymaps）

用 `vim.keymap.set` 來設定快捷鍵：

```lua
vim.keymap.set(模式, 快捷鍵, 動作, 選項)
```

### 模式代碼

| 代碼 | 模式 |
|------|------|
| `"n"` | Normal |
| `"i"` | Insert |
| `"v"` | Visual |
| `"x"` | Visual（不含 Select） |
| `"t"` | Terminal |
| `""` | 所有模式 |

### 實用的基礎映射

```lua
-- 用 jk 快速退出 Insert mode（比按 Esc 方便）
vim.keymap.set("i", "jk", "<Esc>", { desc = "Exit insert mode" })

-- 清除搜尋高亮
vim.keymap.set("n", "<leader>h", ":nohlsearch<CR>", { desc = "Clear search highlight" })

-- 存檔
vim.keymap.set("n", "<leader>w", ":w<CR>", { desc = "Save file" })

-- 離開
vim.keymap.set("n", "<leader>q", ":q<CR>", { desc = "Quit" })
```

### 視窗操作

```lua
-- 分割視窗
vim.keymap.set("n", "<leader>sv", ":vsplit<CR>", { desc = "Split vertical" })
vim.keymap.set("n", "<leader>sh", ":split<CR>", { desc = "Split horizontal" })
vim.keymap.set("n", "<leader>sx", ":close<CR>", { desc = "Close split" })

-- 在視窗間移動（用 Ctrl + hjkl）
vim.keymap.set("n", "<C-h>", "<C-w>h", { desc = "Move to left window" })
vim.keymap.set("n", "<C-j>", "<C-w>j", { desc = "Move to lower window" })
vim.keymap.set("n", "<C-k>", "<C-w>k", { desc = "Move to upper window" })
vim.keymap.set("n", "<C-l>", "<C-w>l", { desc = "Move to right window" })
```

### 行操作

```lua
-- 在 Visual mode 中移動選取的行
vim.keymap.set("v", "J", ":m '>+1<CR>gv=gv", { desc = "Move selection down" })
vim.keymap.set("v", "K", ":m '<-2<CR>gv=gv", { desc = "Move selection up" })

-- 保持游標位置的 J（合併行）
vim.keymap.set("n", "J", "mzJ`z", { desc = "Join lines (keep cursor)" })

-- 翻頁後保持游標在中間
vim.keymap.set("n", "<C-d>", "<C-d>zz", { desc = "Scroll down (centered)" })
vim.keymap.set("n", "<C-u>", "<C-u>zz", { desc = "Scroll up (centered)" })

-- 搜尋跳轉後保持游標在中間
vim.keymap.set("n", "n", "nzzzv", { desc = "Next search (centered)" })
vim.keymap.set("n", "N", "Nzzzv", { desc = "Prev search (centered)" })
```

### Buffer 操作

```lua
-- 切換 buffer（像切換 VSCode 的 tab）
vim.keymap.set("n", "<S-l>", ":bnext<CR>", { desc = "Next buffer" })
vim.keymap.set("n", "<S-h>", ":bprevious<CR>", { desc = "Previous buffer" })
vim.keymap.set("n", "<leader>bd", ":bdelete<CR>", { desc = "Delete buffer" })
```

### 剪貼簿操作

```lua
-- 貼上時不覆蓋 register（在 Visual mode 中）
vim.keymap.set("x", "<leader>p", '"_dP', { desc = "Paste without overwrite" })

-- 刪除到黑洞 register（不影響剪貼簿）
vim.keymap.set("n", "<leader>d", '"_d', { desc = "Delete to void" })
vim.keymap.set("v", "<leader>d", '"_d', { desc = "Delete to void" })
```

## 完整的 init.lua

以下是到目前為止所有設定的完整版本：

```lua
-- ~/.config/nvim/init.lua

-- ========================================
-- Leader Key（必須最先設定）
-- ========================================
vim.g.mapleader = " "
vim.g.maplocalleader = " "

-- ========================================
-- 基本選項
-- ========================================

-- 行號
vim.opt.number = true
vim.opt.relativenumber = true

-- 縮排
vim.opt.tabstop = 4
vim.opt.shiftwidth = 4
vim.opt.expandtab = true
vim.opt.smartindent = true

-- 搜尋
vim.opt.ignorecase = true
vim.opt.smartcase = true
vim.opt.hlsearch = true
vim.opt.incsearch = true

-- 外觀
vim.opt.termguicolors = true
vim.opt.cursorline = true
vim.opt.signcolumn = "yes"
vim.opt.scrolloff = 8
vim.opt.colorcolumn = "80"

-- 系統整合
vim.opt.clipboard = "unnamedplus"
vim.opt.mouse = "a"
vim.opt.undofile = true

-- 視窗
vim.opt.splitbelow = true
vim.opt.splitright = true

-- 其他
vim.opt.wrap = false
vim.opt.swapfile = false
vim.opt.backup = false
vim.opt.updatetime = 250
vim.opt.timeoutlen = 300
vim.opt.completeopt = { "menuone", "noselect" }

-- ========================================
-- 快捷鍵映射
-- ========================================

-- Insert mode
vim.keymap.set("i", "jk", "<Esc>", { desc = "Exit insert mode" })

-- 一般操作
vim.keymap.set("n", "<leader>h", ":nohlsearch<CR>", { desc = "Clear search highlight" })
vim.keymap.set("n", "<leader>w", ":w<CR>", { desc = "Save file" })
vim.keymap.set("n", "<leader>q", ":q<CR>", { desc = "Quit" })

-- 視窗分割
vim.keymap.set("n", "<leader>sv", ":vsplit<CR>", { desc = "Split vertical" })
vim.keymap.set("n", "<leader>sh", ":split<CR>", { desc = "Split horizontal" })
vim.keymap.set("n", "<leader>sx", ":close<CR>", { desc = "Close split" })

-- 視窗移動
vim.keymap.set("n", "<C-h>", "<C-w>h", { desc = "Move to left window" })
vim.keymap.set("n", "<C-j>", "<C-w>j", { desc = "Move to lower window" })
vim.keymap.set("n", "<C-k>", "<C-w>k", { desc = "Move to upper window" })
vim.keymap.set("n", "<C-l>", "<C-w>l", { desc = "Move to right window" })

-- 行移動
vim.keymap.set("v", "J", ":m '>+1<CR>gv=gv", { desc = "Move selection down" })
vim.keymap.set("v", "K", ":m '<-2<CR>gv=gv", { desc = "Move selection up" })
vim.keymap.set("n", "J", "mzJ`z", { desc = "Join lines (keep cursor)" })

-- 滾動保持游標在中間
vim.keymap.set("n", "<C-d>", "<C-d>zz", { desc = "Scroll down (centered)" })
vim.keymap.set("n", "<C-u>", "<C-u>zz", { desc = "Scroll up (centered)" })
vim.keymap.set("n", "n", "nzzzv", { desc = "Next search (centered)" })
vim.keymap.set("n", "N", "Nzzzv", { desc = "Prev search (centered)" })

-- Buffer
vim.keymap.set("n", "<S-l>", ":bnext<CR>", { desc = "Next buffer" })
vim.keymap.set("n", "<S-h>", ":bprevious<CR>", { desc = "Previous buffer" })
vim.keymap.set("n", "<leader>bd", ":bdelete<CR>", { desc = "Delete buffer" })

-- 剪貼簿
vim.keymap.set("x", "<leader>p", '"_dP', { desc = "Paste without overwrite" })
vim.keymap.set("n", "<leader>d", '"_d', { desc = "Delete to void" })
vim.keymap.set("v", "<leader>d", '"_d', { desc = "Delete to void" })
```

## 配置檔的組織方式

現在所有設定都在一個 `init.lua` 裡，很簡單。但當設定越來越多（加入 plugin 之後），你會想把它拆分成多個檔案。

Neovim 約定的目錄結構：

```
~/.config/nvim/
├── init.lua              ← 入口點
├── lua/
│   └── config/
│       ├── options.lua   ← vim.opt 設定
│       ├── keymaps.lua   ← 快捷鍵映射
│       ├── lazy.lua      ← Plugin manager（下一篇）
│       └── autocmds.lua  ← 自動命令
```

在 `init.lua` 中用 `require` 引入：

```lua
require("config.options")
require("config.keymaps")
```

我們在下一篇安裝 plugin manager 時會正式採用這種結構。目前先把所有設定放在 `init.lua` 就好。

## 驗證你的設定

重啟 Neovim（或在 Neovim 中執行 `:source %`），確認以下幾點：

1. **行號**：左邊應該顯示相對行號
2. **空白鍵 + w**：應該可以存檔
3. **空白鍵 + q**：應該可以離開
4. **jk**：在 Insert mode 按 jk 應該回到 Normal mode
5. **Ctrl + d/u**：滾動時游標保持在螢幕中間
6. **系統剪貼簿**：在 Neovim 中 yy 複製，到其他程式中可以 Cmd+V 貼上

如果有任何設定不生效，可以用 `:checkhealth` 來診斷問題。

## 小結

恭喜！你現在有了一個配置完善的 Neovim 基礎環境：

- 舒適的視覺設定（行號、游標行、色彩）
- 智能的搜尋行為
- 方便的 Leader Key 快捷鍵系統
- 與系統剪貼簿的整合
- 持久化的 undo 歷史

不過現在的 Neovim 還很「素」——沒有漂亮的主題、沒有檔案樹、沒有自動補全。這些都需要 **plugins**。

下一篇，我們會安裝 Neovim 最受歡迎的 plugin manager——**lazy.nvim**，並開始安裝你的第一批 plugins。

---

## Neovim 系列文章導航

**基礎篇**
1. [為什麼我想離開 VSCode？](/posts/neovim-series-01-why-leave-vscode/)
2. [Vim 的語言——動詞、名詞、組合技](/posts/neovim-series-02-vim-language/)
3. [移動的藝術](/posts/neovim-series-03-movement/)
4. [編輯效率加倍](/posts/neovim-series-04-editing-power/)

**設定篇**
5. **安裝 Neovim 與 init.lua（本篇）**
6. [Plugin 管理 lazy.nvim](/posts/neovim-series-06-plugin-management/)
7. [程式碼智能 LSP + 補全](/posts/neovim-series-07-lsp-completion/)
8. [語法高亮與檔案導航](/posts/neovim-series-08-treesitter-telescope/)

**進階篇**
9. [Git 整合工作流](/posts/neovim-series-09-git-integration/)
10. [Terminal 與 Claude Code 整合](/posts/neovim-series-10-terminal-claude-code/)
11. [打造快捷鍵系統](/posts/neovim-series-11-keymap-system/)
12. [從 VSCode 完全畢業](/posts/neovim-series-12-graduation/)
