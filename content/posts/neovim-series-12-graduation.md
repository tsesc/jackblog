+++
title = 'Neovim 系列（十二）：從 VSCode 完全畢業——完整配置、痛點解法與學習資源'
date = 2026-03-06T10:00:00+08:00
draft = false
tags = ['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列']
categories = ['技術筆記']
author = 'Jack'
description = '系列完結篇——VSCode 到 Neovim 的完整功能對照表、Top 10 常見痛點與解法、完整配置總覽、以及持續精進的學習資源推薦'
toc = true
weight = 12
+++

> 這是 **Neovim 從零開始**系列的第十二篇，也是最後一篇。整個系列共 12 篇文章，帶你從完全不懂 Vim，到能用 Neovim 打造一個完整的現代化開發環境。

## 恭喜你走到這裡

如果你從第一篇看到這裡，並且實際動手設定了你的 Neovim，那你已經擁有了一個功能完整、高度客製化的開發環境。讓我們回顧一下你已經學會的東西：

| 篇章 | 學會的技能 |
|------|-----------|
| 01 | 理解 modal editing 和 Vim 哲學 |
| 02 | Vim 的語言系統：operator + motion/text object |
| 03 | 完整的移動指令體系 |
| 04 | Registers、Macros、Dot Command |
| 05 | 安裝 Neovim、編寫 init.lua |
| 06 | 用 lazy.nvim 管理 plugins |
| 07 | LSP + 自動補全 |
| 08 | Treesitter + Telescope + 檔案導航 |
| 09 | Git 整合工作流 |
| 10 | Terminal 整合 + Claude Code 工作流 |
| 11 | which-key 快捷鍵系統設計 |
| 12 | 完整配置總覽（你正在看的這篇） |

## VSCode → Neovim 完整功能對照表

這張表是你遷移過程中最重要的參考。當你想做某件事但不知道在 Neovim 中怎麼做時，來這裡查：

### 基本編輯

| 功能 | VSCode | Neovim |
|------|--------|--------|
| 儲存 | `Cmd+S` | `<leader>w` 或 `:w` |
| 復原 | `Cmd+Z` | `u` |
| 重做 | `Cmd+Shift+Z` | `Ctrl-r` |
| 全選 | `Cmd+A` | `ggVG` |
| 剪下行 | `Cmd+X`（空選取） | `dd` |
| 複製行 | `Cmd+C`（空選取） | `yy` |
| 向上/下移動行 | `Alt+↑/↓` | `Visual mode + K/J` |
| 向上/下複製行 | `Alt+Shift+↑/↓` | `yy` + `p`（或 `yyP`） |
| 刪除行 | `Cmd+Shift+K` | `dd` |
| 縮排 | `Tab` / `Shift+Tab` | `>>` / `<<` |
| 切換註解 | `Cmd+/` | `gcc` |
| 多行註解 | `Cmd+Shift+/` | `Visual + gc` |
| 行尾加文字 | `End` + 輸入 | `A` + 輸入 |
| 行首加文字 | `Home` + 輸入 | `I` + 輸入 |

### 導航

| 功能 | VSCode | Neovim |
|------|--------|--------|
| 搜尋檔案 | `Cmd+P` | `<leader>ff` |
| 全域搜尋 | `Cmd+Shift+F` | `<leader>fg` |
| 搜尋符號 | `Cmd+Shift+O` | `<leader>fs` |
| 跳到行號 | `Ctrl+G` | `:{number}` 或 `{number}G` |
| 跳轉到定義 | `F12` / `Cmd+Click` | `gd` |
| 查看引用 | `Shift+F12` | `gr` |
| 返回上一位置 | `Alt+←` | `Ctrl-o` |
| 前往下一位置 | `Alt+→` | `Ctrl-i` |
| 切換檔案 | `Cmd+Tab` | `<leader>fb` 或 `<S-h>`/`<S-l>` |
| 側邊欄 | `Cmd+B` | `-` (oil.nvim) |
| 配對括號跳轉 | — | `%` |

### 程式碼智能

| 功能 | VSCode | Neovim |
|------|--------|--------|
| 自動補全 | 自動觸發 | 自動 / `<C-Space>` |
| 參數提示 | 自動 | `<C-k>` |
| Hover 文件 | 滑鼠懸停 | `K` |
| 快速修復 | `Cmd+.` | `<leader>ca` |
| 重新命名 | `F2` | `<leader>rn` |
| 格式化 | `Shift+Alt+F` | `<leader>f` |
| 上/下一個錯誤 | `F8` | `]d` / `[d` |

### Git

| 功能 | VSCode | Neovim |
|------|--------|--------|
| Source Control 面板 | `Cmd+Shift+G` | `<leader>lg` (lazygit) |
| Stage 檔案 | 點擊 `+` | lazygit `Space` |
| Stage hunk | 行內按鈕 | `<leader>hs` |
| Git Blame | GitLens | `<leader>hb` |
| 查看 Diff | Diff Editor | `<leader>gd` |
| 檔案歷史 | GitLens | `<leader>gh` |

### 視窗管理

| 功能 | VSCode | Neovim |
|------|--------|--------|
| 垂直分割 | `Cmd+\` | `<leader>sv` |
| 切換分割視窗 | `Cmd+1/2/3` | `<C-h/j/k/l>` |
| 關閉分割 | `Cmd+W` | `<leader>sx` |
| Terminal | `` Ctrl+` `` | `<C-\>` |

## Top 10 常見痛點與解法

### 1. 忘記快捷鍵

**解法**：按 `<leader>` 然後等待，which-key 會顯示所有選項。用 `<leader>fk` 搜尋快捷鍵。

### 2. 不小心按了什麼，畫面亂掉了

**解法**：按 `<Esc>` 回到 Normal mode，按 `u` 復原。如果視窗亂了，按 `<C-w>=` 等化視窗大小，或 `:only` 只保留當前視窗。

### 3. 搜尋替換不如 VSCode 直覺

**解法**：
```vim
" 基本替換（整個檔案）
:%s/old/new/g

" 帶確認的替換
:%s/old/new/gc

" 只替換選取範圍：先 Visual 選取，再
:'<,'>s/old/new/g
```

搭配 Telescope 的 `<leader>fg` + quickfix list 可以做到跨檔案搜尋替換。

### 4. 想要多游標編輯

**解法**：Vim 不支援多游標，但有更好的替代方案：
- **Dot Command**：修改一個 → `.` 重複到下一個
- **Macros**：錄製操作 → `@a` 批次執行
- **`:%s`**：搜尋替換
- **Visual Block**：`Ctrl-v` 矩形選取後 `I` 或 `A` 批次插入

這些組合起來比多游標更強大。

### 5. 中文輸入時按 Esc 會切換輸入法

**解法**：安裝 `im-select` 工具，配合設定讓離開 Insert mode 時自動切回英文：

```bash
brew install im-select
```

```lua
vim.api.nvim_create_autocmd("InsertLeave", {
  callback = function()
    vim.fn.system("im-select com.apple.keylayout.ABC")
  end,
})
```

### 6. 複製貼上和系統剪貼簿不同步

**解法**：確認設定中有 `vim.opt.clipboard = "unnamedplus"`。如果還是不行，執行 `:checkhealth` 檢查 clipboard provider。

### 7. 找不到檔案樹/側邊欄

**解法**：按 `-` 打開 oil.nvim（或用 neo-tree 的 `<leader>e`）。Telescope 的 `<leader>ff` 更適合搜尋特定檔案。

### 8. 不會退出 Neovim

**解法**：
| 指令 | 功能 |
|------|------|
| `:q` | 離開（需要先存檔） |
| `:q!` | 不存檔直接離開 |
| `:wq` | 存檔並離開 |
| `ZZ` | 存檔並離開（快捷版） |
| `ZQ` | 不存檔離開（快捷版） |

### 9. Plugin 裝太多，啟動變慢

**解法**：
1. 用 `:Lazy` 按 `P` 查看每個 plugin 的載入時間
2. 確保大多數 plugin 都使用 lazy-loading
3. 移除不常用的 plugin
4. 目標：啟動時間 < 100ms

### 10. 設定檔出錯，Neovim 無法正常啟動

**解法**：用安全模式啟動：
```bash
nvim --clean        # 完全不載入設定
nvim -u NONE        # 不載入 init.lua
```

然後逐步排查設定檔中的問題。

## 完整 Plugin 清單

以下是本系列安裝的所有 plugins 總覽：

| Plugin | 用途 | 篇章 |
|--------|------|------|
| lazy.nvim | Plugin 管理 | 06 |
| catppuccin | 配色主題 | 06 |
| lualine.nvim | 狀態列 | 06 |
| nvim-web-devicons | 檔案圖示 | 06 |
| indent-blankline.nvim | 縮排指引線 | 06 |
| nvim-autopairs | 自動配對括號 | 06 |
| Comment.nvim | 註解快捷鍵 | 06 |
| nvim-lspconfig | LSP 設定 | 07 |
| mason.nvim | LSP server 管理 | 07 |
| mason-lspconfig.nvim | Mason + LSP 橋接 | 07 |
| nvim-cmp | 自動補全引擎 | 07 |
| cmp-nvim-lsp | LSP 補全來源 | 07 |
| cmp-buffer | Buffer 補全來源 | 07 |
| cmp-path | 路徑補全來源 | 07 |
| LuaSnip | Snippet 引擎 | 07 |
| friendly-snippets | Snippet 集合 | 07 |
| nvim-treesitter | 語法高亮 | 08 |
| treesitter-textobjects | 語法結構操作 | 08 |
| telescope.nvim | 模糊搜尋 | 08 |
| telescope-fzf-native | 搜尋加速 | 08 |
| oil.nvim | 檔案瀏覽器 | 08 |
| gitsigns.nvim | Git 行內標記 | 09 |
| lazygit.nvim | lazygit 整合 | 09 |
| diffview.nvim | Diff 檢視 | 09 |
| toggleterm.nvim | Terminal 管理 | 10 |
| which-key.nvim | 快捷鍵提示 | 11 |

## 持續學習資源

### 互動式學習

| 資源 | 說明 |
|------|------|
| `vimtutor` | Neovim 內建教學（在 terminal 輸入 `vimtutor`） |
| Vim Adventures | 用遊戲方式學 Vim（vim-adventures.com） |
| OpenVim | 線上互動式 Vim 教學 |

### 文件

| 資源 | 說明 |
|------|------|
| `:help` | Neovim 內建的完整文件（用 Telescope `<leader>fh` 搜尋） |
| neovim.io/doc | Neovim 官方文件 |
| lazy.folke.io | lazy.nvim 官方文件 |

### 社群

| 資源 | 說明 |
|------|------|
| r/neovim | Reddit Neovim 社群 |
| Neovim Discourse | 官方論壇 |
| GitHub Discussions | Neovim GitHub 討論區 |

### 參考配置

學習別人的配置是進步最快的方式：

| 配置 | 說明 |
|------|------|
| kickstart.nvim | 官方推薦的入門配置 |
| LazyVim | 功能齊全的預配置方案 |
| AstroNvim | 另一個流行的預配置方案 |
| NvChad | 美觀的預配置方案 |

> **注意**：建議先自己從零配置（像本系列這樣），理解每個設定的用途後，再參考這些預配置方案。直接用預配置方案會讓你「會用但不懂」。

### YouTube 頻道

| 頻道 | 風格 |
|------|------|
| ThePrimeagen | 高能量、Vim 技巧 |
| TJ DeVries | Neovim 核心開發者，深度教學 |
| typecraft | Neovim 配置教學 |

## 接下來可以探索的方向

你的 Neovim 配置會隨著時間不斷進化。以下是一些可以繼續探索的方向：

### 進階 Plugins

| Plugin | 用途 |
|--------|------|
| nvim-dap | Debug Adapter Protocol（除錯器） |
| neotest | 測試框架整合 |
| harpoon | 快速切換常用檔案 |
| flash.nvim | 更快的跳轉（取代 f/t） |
| mini.nvim | 一系列小巧的工具集 |
| noice.nvim | 美化訊息和命令列 |

### 工作流優化

- **tmux 進階配置**：session 自動化、佈局保存
- **dotfiles 管理**：用 Git 管理你的所有配置（stow、chezmoi）
- **語言特定配置**：針對你常用的語言做更細緻的 LSP 設定

### Vim 進階技巧

- **`:g` 和 `:v` 全域命令**：對匹配的行執行批次操作
- **`:norm` 命令**：對選取範圍的每一行執行 Normal mode 操作
- **Vim regex**：Vim 的正則表達式有自己的語法
- **Auto commands**：根據事件自動執行操作

## 最後的建議

### 1. 不要追求完美配置

配置是一個持續的過程，不是一次完成的任務。每天改善一點就好。

### 2. 先會用，再優化

不要在配置上花太多時間而忘了寫程式。功能夠用就好，遇到痛點再去改善。

### 3. 享受過程

學 Vim/Neovim 的過程本身就很有趣。每天發現一個新的快捷操作、一個新的 plugin、一個更好的工作流——這就是 Vim 使用者樂此不疲的原因。

### 4. 你不需要放棄 VSCode

Neovim 和 VSCode 不是非此即彼的選擇。你可以在不同場景下使用不同工具。但我相信，一旦你習慣了 Neovim 的工作流，你會越來越少打開 VSCode。

歡迎來到 Neovim 的世界。享受這趟旅程！

---

## Neovim 系列文章導航

**基礎篇**
1. [為什麼我想離開 VSCode？](/posts/neovim-series-01-why-leave-vscode/)
2. [Vim 的語言——動詞、名詞、組合技](/posts/neovim-series-02-vim-language/)
3. [移動的藝術](/posts/neovim-series-03-movement/)
4. [編輯效率加倍](/posts/neovim-series-04-editing-power/)

**設定篇**
5. [安裝 Neovim 與 init.lua](/posts/neovim-series-05-install-and-init-lua/)
6. [Plugin 管理 lazy.nvim](/posts/neovim-series-06-plugin-management/)
7. [程式碼智能 LSP + 補全](/posts/neovim-series-07-lsp-completion/)
8. [語法高亮與檔案導航](/posts/neovim-series-08-treesitter-telescope/)

**進階篇**
9. [Git 整合工作流](/posts/neovim-series-09-git-integration/)
10. [Terminal 與 Claude Code 整合](/posts/neovim-series-10-terminal-claude-code/)
11. [打造快捷鍵系統](/posts/neovim-series-11-keymap-system/)
12. **從 VSCode 完全畢業（本篇）**
