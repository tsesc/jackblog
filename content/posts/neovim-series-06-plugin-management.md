+++
title = 'Neovim 系列（六）：Plugin 管理——用 lazy.nvim 打造你的工具箱'
date = 2026-02-28T10:00:00+08:00
draft = false
tags = ['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列', 'lazy.nvim']
categories = ['技術筆記']
author = 'Jack'
description = '學習使用 lazy.nvim 管理 Neovim plugins——安裝、設定、lazy-loading 概念，並安裝第一批實用 plugins：配色主題、狀態列、圖示'
toc = true
weight = 6
+++

> 這是 **Neovim 從零開始**系列的第六篇。整個系列共 12 篇文章，將帶你從完全不懂 Vim，到能用 Neovim 打造一個完整的現代化開發環境。

## 為什麼需要 Plugins？

上一篇我們建立了基礎的 `init.lua`，但 Neovim 現在看起來還很陽春。要讓它成為一個真正好用的開發環境，我們需要 plugins 來擴充功能：

- 漂亮的配色主題
- 狀態列顯示檔案資訊
- 檔案瀏覽器
- 模糊搜尋
- 語法高亮
- 自動補全
- Git 整合

這些都是 plugins 提供的。而要管理這些 plugins，我們需要一個 **plugin manager**。

## 認識 lazy.nvim

**lazy.nvim** 是目前 Neovim 最主流的 plugin manager。它的優點：

- **速度極快**：自動 lazy-loading，啟動時間極短
- **直覺的 UI**：按 `:Lazy` 就能看到所有 plugin 的狀態
- **宣告式配置**：用 Lua table 定義 plugin，清楚明瞭
- **自動管理依賴**：plugin 之間的依賴關係自動處理
- **鎖定版本**：`lazy-lock.json` 確保可重現的配置

### 重新組織配置結構

在安裝 lazy.nvim 之前，先把配置拆分成更好的結構：

```
~/.config/nvim/
├── init.lua                  ← 入口點
├── lua/
│   └── config/
│       ├── options.lua       ← vim.opt 設定
│       ├── keymaps.lua       ← 快捷鍵映射
│       └── lazy.lua          ← lazy.nvim 初始化
│   └── plugins/
│       ├── colorscheme.lua   ← 配色主題
│       ├── lualine.lua       ← 狀態列
│       └── ...               ← 其他 plugins
```

### 拆分現有設定

把上一篇 `init.lua` 的內容拆分：

**`lua/config/options.lua`**：

```lua
-- Leader Key（必須最先設定）
vim.g.mapleader = " "
vim.g.maplocalleader = " "

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
```

**`lua/config/keymaps.lua`**：

```lua
vim.keymap.set("i", "jk", "<Esc>", { desc = "Exit insert mode" })
vim.keymap.set("n", "<leader>h", ":nohlsearch<CR>", { desc = "Clear search highlight" })
vim.keymap.set("n", "<leader>w", ":w<CR>", { desc = "Save file" })
vim.keymap.set("n", "<leader>q", ":q<CR>", { desc = "Quit" })

-- 視窗
vim.keymap.set("n", "<leader>sv", ":vsplit<CR>", { desc = "Split vertical" })
vim.keymap.set("n", "<leader>sh", ":split<CR>", { desc = "Split horizontal" })
vim.keymap.set("n", "<leader>sx", ":close<CR>", { desc = "Close split" })
vim.keymap.set("n", "<C-h>", "<C-w>h", { desc = "Move to left window" })
vim.keymap.set("n", "<C-j>", "<C-w>j", { desc = "Move to lower window" })
vim.keymap.set("n", "<C-k>", "<C-w>k", { desc = "Move to upper window" })
vim.keymap.set("n", "<C-l>", "<C-w>l", { desc = "Move to right window" })

-- 行移動
vim.keymap.set("v", "J", ":m '>+1<CR>gv=gv", { desc = "Move selection down" })
vim.keymap.set("v", "K", ":m '<-2<CR>gv=gv", { desc = "Move selection up" })
vim.keymap.set("n", "J", "mzJ`z", { desc = "Join lines (keep cursor)" })

-- 滾動
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

**`init.lua`**（新的入口點）：

```lua
require("config.options")
require("config.keymaps")
require("config.lazy")
```

## 安裝 lazy.nvim

建立 `lua/config/lazy.lua`：

```lua
-- Bootstrap lazy.nvim（自動安裝）
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable",
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- 設定 lazy.nvim
require("lazy").setup("plugins", {
  checker = {
    enabled = true,     -- 自動檢查 plugin 更新
    notify = false,     -- 不要彈出通知
  },
  change_detection = {
    notify = false,     -- 設定檔變更時不通知
  },
})
```

這段程式碼做了兩件事：
1. **Bootstrap**：如果 lazy.nvim 還沒安裝，自動從 GitHub clone
2. **Setup**：告訴 lazy.nvim 去 `lua/plugins/` 目錄載入所有 plugin 定義

## Plugin Spec 格式

lazy.nvim 用 Lua table 來定義每個 plugin。基本格式：

```lua
-- lua/plugins/example.lua
return {
  "github-user/plugin-name",   -- GitHub repo 路徑
  dependencies = { ... },       -- 依賴的其他 plugin
  config = function()           -- plugin 載入後執行的設定
    require("plugin-name").setup({
      -- 設定選項
    })
  end,
  event = "VeryLazy",          -- 何時載入（lazy-loading）
  keys = { ... },              -- 綁定快捷鍵時載入
  cmd = { "CommandName" },     -- 執行指定命令時載入
}
```

每個 `lua/plugins/*.lua` 檔案 return 一個 table（一個 plugin）或一個 table 的 array（多個 plugins）。

## 安裝第一批 Plugins

### 1. 配色主題：catppuccin

```lua
-- lua/plugins/colorscheme.lua
return {
  "catppuccin/nvim",
  name = "catppuccin",
  priority = 1000,  -- 確保最先載入
  config = function()
    require("catppuccin").setup({
      flavour = "mocha",  -- latte, frappe, macchiato, mocha
      integrations = {
        treesitter = true,
        telescope = { enabled = true },
        gitsigns = true,
        which_key = true,
      },
    })
    vim.cmd.colorscheme("catppuccin")
  end,
}
```

Catppuccin 是目前最受歡迎的 Neovim 主題之一，有四種風格：
- **Latte**：亮色主題
- **Frappe**：淺色暗色主題
- **Macchiato**：中等暗色主題
- **Mocha**：深色主題（最受歡迎）

其他熱門主題：`tokyonight.nvim`、`gruvbox.nvim`、`rose-pine`。

### 2. 狀態列：lualine.nvim

```lua
-- lua/plugins/lualine.lua
return {
  "nvim-lualine/lualine.nvim",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  config = function()
    require("lualine").setup({
      options = {
        theme = "catppuccin",
        component_separators = { left = "", right = "" },
        section_separators = { left = "", right = "" },
      },
      sections = {
        lualine_a = { "mode" },
        lualine_b = { "branch", "diff", "diagnostics" },
        lualine_c = { "filename" },
        lualine_x = { "encoding", "fileformat", "filetype" },
        lualine_y = { "progress" },
        lualine_z = { "location" },
      },
    })
  end,
}
```

Lualine 在畫面底部顯示一個資訊豐富的狀態列：目前模式、Git 分支、檔案名稱、檔案類型、行列位置等。

### 3. 檔案圖示：nvim-web-devicons

```lua
-- lua/plugins/devicons.lua
return {
  "nvim-tree/nvim-web-devicons",
  lazy = true,  -- 被其他 plugin 依賴時才載入
}
```

這個 plugin 提供檔案類型圖示（需要 Nerd Font）。

> **安裝 Nerd Font**：
> ```bash
> brew install --cask font-hack-nerd-font
> ```
> 安裝後在 terminal 的設定中選用這個字型。

### 4. 縮排指引線：indent-blankline

```lua
-- lua/plugins/indent-blankline.lua
return {
  "lukas-reineke/indent-blankline.nvim",
  main = "ibl",
  event = "BufReadPost",
  config = function()
    require("ibl").setup({
      indent = { char = "│" },
      scope = { enabled = true },
    })
  end,
}
```

在每一層縮排處顯示垂直線，讓程式碼的層次結構一目了然。

### 5. 括號配對高亮

```lua
-- lua/plugins/autopairs.lua
return {
  "windwp/nvim-autopairs",
  event = "InsertEnter",
  config = function()
    require("nvim-autopairs").setup({})
  end,
}
```

自動補全括號、引號等成對符號。輸入 `(` 時自動補上 `)`。

### 6. 註解快捷鍵：Comment.nvim

```lua
-- lua/plugins/comment.lua
return {
  "numToStr/Comment.nvim",
  event = "BufReadPost",
  config = function()
    require("Comment").setup()
  end,
}
```

安裝後：
- `gcc` — 切換當前行的註解
- `gc` + motion — 切換指定範圍的註解（如 `gcap` 註解整個段落）
- Visual mode 選取後按 `gc` — 切換多行註解

對照 VSCode 的 `Cmd+/`，Vim 的方式更靈活。

## Lazy-Loading 概念

你可能注意到上面的 plugin 設定中有 `event`、`cmd`、`keys` 等欄位。這就是 **lazy-loading**——讓 plugin 只在需要的時候才載入，大幅加快啟動速度。

| 觸發條件 | 範例 | 用途 |
|---------|------|------|
| `event` | `"BufReadPost"` | 開啟檔案時載入 |
| `event` | `"InsertEnter"` | 進入 Insert mode 時載入 |
| `event` | `"VeryLazy"` | 啟動完成後延遲載入 |
| `cmd` | `"Telescope"` | 執行 `:Telescope` 命令時載入 |
| `keys` | `"<leader>ff"` | 按下快捷鍵時載入 |
| `ft` | `"lua"` | 開啟指定檔案類型時載入 |
| `lazy = true` | — | 只在被其他 plugin 依賴時載入 |

**原則**：
- 配色主題用 `priority = 1000`，確保最先載入
- UI 相關的 plugin 用 `event = "VeryLazy"`
- 編輯增強用 `event = "BufReadPost"` 或 `"InsertEnter"`
- 不常用的工具用 `cmd` 或 `keys`

## 使用 Lazy UI

安裝完成後，在 Neovim 中按 `:Lazy` 就能打開管理介面：

| 快捷鍵 | 功能 |
|--------|------|
| `I` | 安裝缺少的 plugins |
| `U` | 更新所有 plugins |
| `S` | 同步（安裝 + 更新 + 清除） |
| `C` | 清除不再使用的 plugins |
| `P` | 查看 plugin 效能（載入時間） |
| `L` | 查看 log |
| `q` | 關閉 |

按 `P` 可以看到每個 plugin 的載入時間，幫助你優化啟動速度。

## 驗證安裝

重啟 Neovim 後，lazy.nvim 會自動下載並安裝所有 plugins。確認以下幾點：

1. **配色主題**：畫面應該有漂亮的 Catppuccin 配色
2. **狀態列**：底部有 lualine 顯示模式、分支、檔案等資訊
3. **縮排線**：打開程式碼檔案，應該看到縮排指引線
4. **自動配對**：在 Insert mode 輸入 `(` 應該自動補上 `)`
5. **註解**：在 Normal mode 按 `gcc` 可以切換註解

## 小結

現在你的 Neovim 已經從「素面朝天」變成一個有模有樣的編輯器了：

- **lazy.nvim** 管理所有 plugins
- **catppuccin** 提供漂亮的配色
- **lualine** 顯示豐富的狀態資訊
- **indent-blankline** 讓程式碼層次清晰
- **nvim-autopairs** 自動補全括號
- **Comment.nvim** 提供方便的註解快捷鍵

但作為一個開發環境，我們還缺少最關鍵的功能——**程式碼智能**。下一篇，我們會設定 LSP（Language Server Protocol）和自動補全，讓 Neovim 擁有和 VSCode 一樣的程式碼導航和補全能力。
