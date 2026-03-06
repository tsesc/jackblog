+++
title = 'Neovim 系列（十）：Terminal 與 Claude Code 整合——AI 時代的開發工作流'
date = 2026-03-04T10:00:00+08:00
draft = false
tags = ['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列', 'Claude Code', 'Terminal']
categories = ['技術筆記']
author = 'Jack'
description = '學習 Neovim 的 Terminal Mode、toggleterm.nvim 的使用，以及如何結合 Claude Code 打造高效的 AI 輔助開發工作流——一切都在 terminal 中完成'
toc = true
weight = 10
+++

> 這是 **Neovim 從零開始**系列的第十篇。整個系列共 12 篇文章，將帶你從完全不懂 Vim，到能用 Neovim 打造一個完整的現代化開發環境。

## Terminal 工作流的魅力

這是我當初想從 VSCode 轉到 Neovim 的核心原因之一：**讓整個開發流程都在 terminal 中完成**。

想像一下這樣的畫面：

```
┌──────────────────────────────────────┐
│ tmux session: my-project             │
│ ┌────────────────┬─────────────────┐ │
│ │                │                 │ │
│ │   Neovim       │  Claude Code    │ │
│ │   (寫程式)      │  (AI 助手)      │ │
│ │                │                 │ │
│ ├────────────────┴─────────────────┤ │
│ │    Shell (跑測試、git 操作)       │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

所有工作在同一個 terminal 視窗中完成。不切換應用程式、不碰滑鼠、不離開鍵盤。

## Neovim 的 Terminal Mode

Neovim 內建了 Terminal emulator——你可以在 Neovim 的 buffer 中直接跑 shell。

### 基本操作

```vim
:terminal           " 在當前視窗開啟 terminal
:split | terminal   " 在下方分割開啟 terminal
:vsplit | terminal  " 在右側分割開啟 terminal
```

### Terminal Mode 的模式切換

Terminal 有自己的模式：

| 狀態 | 說明 | 進入方式 |
|------|------|---------|
| Terminal mode | 像普通 terminal 一樣使用 | 自動（打開 terminal 時） |
| Normal mode | 可以用 Vim 操作查看 terminal 輸出 | `<C-\><C-n>` |

**最重要的快捷鍵**：`Ctrl-\ Ctrl-n` — 從 Terminal mode 回到 Normal mode。

這讓你可以用 Vim 的方式操作 terminal 輸出：搜尋、複製、跳轉等。

### 基本的 Terminal 快捷鍵設定

```lua
-- 在 terminal 中用 Esc 或 jk 回到 Normal mode
vim.keymap.set("t", "<Esc>", "<C-\\><C-n>", { desc = "Exit terminal mode" })
vim.keymap.set("t", "jk", "<C-\\><C-n>", { desc = "Exit terminal mode" })

-- 在 terminal 中用 Ctrl+hjkl 切換視窗
vim.keymap.set("t", "<C-h>", "<C-\\><C-n><C-w>h", { desc = "Move to left window" })
vim.keymap.set("t", "<C-j>", "<C-\\><C-n><C-w>j", { desc = "Move to lower window" })
vim.keymap.set("t", "<C-k>", "<C-\\><C-n><C-w>k", { desc = "Move to upper window" })
vim.keymap.set("t", "<C-l>", "<C-\\><C-n><C-w>l", { desc = "Move to right window" })
```

## toggleterm.nvim：更好的 Terminal 體驗

內建 terminal 功能完整但使用起來有點生硬。**toggleterm.nvim** 提供了更好的體驗：

```lua
-- lua/plugins/toggleterm.lua
return {
  "akinsho/toggleterm.nvim",
  version = "*",
  config = function()
    require("toggleterm").setup({
      size = function(term)
        if term.direction == "horizontal" then
          return 15
        elseif term.direction == "vertical" then
          return vim.o.columns * 0.4
        end
      end,
      open_mapping = [[<C-\>]],
      hide_numbers = true,
      shade_terminals = true,
      start_in_insert = true,
      insert_mappings = true,
      terminal_mappings = true,
      persist_size = true,
      direction = "horizontal",  -- horizontal, vertical, float, tab
      close_on_exit = true,
      float_opts = {
        border = "curved",
        width = math.floor(vim.o.columns * 0.8),
        height = math.floor(vim.o.lines * 0.8),
      },
    })

    -- 自訂 terminal 快捷鍵
    vim.keymap.set("n", "<leader>th", "<cmd>ToggleTerm direction=horizontal<cr>",
      { desc = "Toggle horizontal terminal" })
    vim.keymap.set("n", "<leader>tv", "<cmd>ToggleTerm direction=vertical<cr>",
      { desc = "Toggle vertical terminal" })
    vim.keymap.set("n", "<leader>tf", "<cmd>ToggleTerm direction=float<cr>",
      { desc = "Toggle float terminal" })
  end,
}
```

### toggleterm 的操作

| 快捷鍵 | 功能 |
|--------|------|
| `Ctrl-\` | 切換 terminal（開/關） |
| `<leader>th` | 開啟水平 terminal |
| `<leader>tv` | 開啟垂直 terminal |
| `<leader>tf` | 開啟浮動 terminal |
| `2<C-\>` | 開啟第 2 個 terminal（可以有多個） |

### 多個 Terminal

toggleterm 支援多個編號的 terminal：

```
1<C-\>  → Terminal #1
2<C-\>  → Terminal #2
3<C-\>  → Terminal #3
```

你可以用一個跑測試、一個跑 server、一個跑 Claude Code。

## 與 Claude Code 的工作流

這是本篇的重頭戲。Claude Code 是一個 terminal-native 的 AI 開發工具，和 Neovim 的搭配天衣無縫。

### 方案一：tmux 分割（推薦）

最穩定也最靈活的方式是用 **tmux** 來管理多個 terminal：

```bash
# 安裝 tmux
brew install tmux
```

基本的 tmux 工作流：

```
# 開啟 tmux session
tmux new -s myproject

# 垂直分割（左邊 Neovim，右邊 Claude Code）
# Ctrl-b %

# 左邊跑 Neovim
nvim .

# 右邊跑 Claude Code
claude
```

tmux 常用快捷鍵（預設前綴是 `Ctrl-b`）：

| 快捷鍵 | 功能 |
|--------|------|
| `Ctrl-b %` | 垂直分割 |
| `Ctrl-b "` | 水平分割 |
| `Ctrl-b ←/→` | 在 pane 間切換 |
| `Ctrl-b z` | 最大化/還原目前 pane |
| `Ctrl-b d` | Detach（背景執行） |
| `tmux a` | 重新連接到 session |

### 方案二：Neovim Terminal 分割

用 Neovim 內建的 terminal 或 toggleterm 在 Neovim 內開啟 Claude Code：

```lua
-- 開啟一個專門給 Claude Code 的浮動 terminal
vim.keymap.set("n", "<leader>cc", function()
  local Terminal = require("toggleterm.terminal").Terminal
  local claude = Terminal:new({
    cmd = "claude",
    direction = "float",
    close_on_exit = false,
  })
  claude:toggle()
end, { desc = "Toggle Claude Code" })
```

### 方案三：獨立的 Terminal Tab

在 terminal app（如 iTerm2、WezTerm）中開多個 tab：

- Tab 1：Neovim
- Tab 2：Claude Code
- Tab 3：Shell（跑測試等）

用 `Cmd+1`/`Cmd+2`/`Cmd+3` 切換。

### 推薦的工作流程

以下是我個人最常用的 Neovim + Claude Code 工作流程：

```
┌─────── tmux session ──────────────┐
│                                    │
│  [Pane 1: Neovim]  [Pane 2: Shell] │
│  ┌──────────────┐  ┌────────────┐ │
│  │              │  │            │ │
│  │  編輯程式碼   │  │ Claude Code│ │
│  │              │  │ 或         │ │
│  │              │  │ 跑測試     │ │
│  │              │  │            │ │
│  └──────────────┘  └────────────┘ │
│                                    │
└────────────────────────────────────┘
```

**典型的開發循環**：

1. **在 Neovim 中寫程式碼**
2. **切到右側 pane，用 Claude Code 詢問問題或請求協助**
3. **Claude Code 修改了檔案** → Neovim 自動重載（可以設定 autoread）
4. **在 Neovim 中檢視 Claude 的修改** → 用 `]h` `[h` 查看 git diff
5. **用 gitsigns stage/reset** 需要或不需要的變更
6. **重複**

### 自動重載設定

Claude Code 可能會在背景修改你的檔案，加上這個設定讓 Neovim 自動重載：

```lua
-- 加在 options.lua 中
vim.opt.autoread = true

-- 自動命令：當 Neovim 獲得焦點或切換 buffer 時檢查檔案變更
vim.api.nvim_create_autocmd({ "FocusGained", "BufEnter", "CursorHold" }, {
  command = "checktime",
})
```

## 進階 Terminal 技巧

### 發送命令到 Terminal

你可以從 Neovim 直接發送命令到 terminal，不需要手動切換：

```lua
-- 把當前行發送到 terminal 執行
vim.keymap.set("n", "<leader>tl", function()
  local line = vim.api.nvim_get_current_line()
  require("toggleterm").exec(line)
end, { desc = "Send line to terminal" })
```

### 快速執行常用命令

```lua
-- 跑測試
vim.keymap.set("n", "<leader>tt", function()
  require("toggleterm").exec("npm test")
end, { desc = "Run tests" })

-- 跑當前檔案（Go）
vim.keymap.set("n", "<leader>tr", function()
  local file = vim.fn.expand("%")
  require("toggleterm").exec("go run " .. file)
end, { desc = "Run current file" })
```

## Terminal Multiplexer：tmux 基礎

如果你打算長期使用 terminal 開發環境，學習 tmux 是非常值得的投資。以下是最實用的功能：

### Session 管理

```bash
tmux new -s project-name    # 建立新 session
tmux ls                     # 列出所有 session
tmux a -t project-name      # 連接到 session
tmux kill-session -t name   # 關閉 session
```

Session 的好處：即使你關閉 terminal 視窗，tmux session 仍在背景執行。下次打開 terminal 用 `tmux a` 就能回到原來的工作狀態。

### 建議的 tmux 配置

```bash
# ~/.tmux.conf

# 把前綴改成 Ctrl-a（比 Ctrl-b 好按）
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# 啟用滑鼠
set -g mouse on

# 從 1 開始編號
set -g base-index 1
setw -g pane-base-index 1

# 256 色支援
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"

# Vim 風格的 pane 切換
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# 快速重載設定
bind r source-file ~/.tmux.conf \; display "Reloaded!"
```

## 與 VSCode Terminal 的對照

| 功能 | VSCode | Neovim + tmux |
|------|--------|--------------|
| 內建 terminal | `` Ctrl+` `` | `<C-\>` (toggleterm) |
| 分割 terminal | 內建支援 | tmux pane / Neovim split |
| 多個 terminal | Tab 式管理 | toggleterm 編號 / tmux windows |
| 背景執行 | 關閉 VSCode 就結束 | tmux session 持續執行 |
| SSH 遠端 | Remote SSH 擴充 | 原生支援（tmux + ssh） |
| AI 整合 | Copilot 面板 | Claude Code 在旁邊的 pane |

Neovim + tmux 的組合在 **SSH 遠端開發** 和 **背景持續運行** 方面遠勝 VSCode。

## 小結

本篇介紹了三種 terminal 整合方式：

| 方式 | 優點 | 適合場景 |
|------|------|---------|
| Neovim 內建 terminal | 不需額外工具 | 簡單任務 |
| toggleterm.nvim | 方便切換、支援浮動視窗 | 日常開發 |
| tmux | 最靈活、session 持久化 | 專業開發環境 |

搭配 Claude Code 的工作流核心思想是：**把編輯器和 AI 助手放在同一個 terminal 環境中**，用鍵盤快捷鍵在它們之間無縫切換。

下一篇，我們要來設計整個快捷鍵系統——用 **which-key** 把所有功能組織成一個有邏輯、易記憶的鍵位架構。
