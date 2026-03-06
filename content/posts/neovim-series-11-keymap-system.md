+++
title = 'Neovim 系列（十一）：打造你的快捷鍵系統——which-key 與 Leader Key 設計哲學'
date = 2026-03-05T10:00:00+08:00
draft = false
tags = ['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列', 'which-key']
categories = ['技術筆記']
author = 'Jack'
description = '用 which-key.nvim 建立可探索的快捷鍵系統，學習 Leader Key 的設計哲學，把所有功能按邏輯分組——讓你不用背快捷鍵也能找到需要的功能'
toc = true
weight = 11
+++

> 這是 **Neovim 從零開始**系列的第十一篇。整個系列共 12 篇文章，將帶你從完全不懂 Vim，到能用 Neovim 打造一個完整的現代化開發環境。

## 快捷鍵的困境

到目前為止，我們已經設定了大量的快捷鍵：LSP 操作、Telescope 搜尋、Git 操作、Terminal 切換... 如果你覺得開始記不住了，這很正常。

解決方案不是「全部背下來」，而是用一套**有邏輯的系統**來組織它們，加上一個**可探索的提示工具**。

這就是 **which-key.nvim** 的用處。

## which-key.nvim：按鍵提示器

which-key 的功能很簡單：當你按下一個前綴鍵（如 `<leader>`）後**稍微等一下**，它會彈出一個面板，顯示所有可用的下一步按鍵和它們的功能。

```
按下 <Space>（leader key）後等待...

┌─────────────────────────────────┐
│ ╭──── Leader Key ─────────────╮ │
│ │ b → Buffers                 │ │
│ │ c → Code actions            │ │
│ │ d → Delete to void          │ │
│ │ e → Diagnostics             │ │
│ │ f → Find (Telescope)        │ │
│ │ g → Git                     │ │
│ │ h → Hunk (Git changes)      │ │
│ │ l → LSP                     │ │
│ │ q → Quit                    │ │
│ │ s → Split                   │ │
│ │ t → Terminal                │ │
│ │ w → Save                    │ │
│ ╰─────────────────────────────╯ │
└─────────────────────────────────┘
```

你不需要記住所有快捷鍵——只要記住前綴的邏輯分組，其他的讓 which-key 提醒你。

### 安裝 which-key

```lua
-- lua/plugins/which-key.lua
return {
  "folke/which-key.nvim",
  event = "VeryLazy",
  config = function()
    local wk = require("which-key")
    wk.setup({
      plugins = {
        marks = true,
        registers = true,
        spelling = { enabled = false },
      },
      win = {
        border = "rounded",
      },
      layout = {
        align = "center",
      },
    })

    -- 註冊 Leader Key 的群組名稱
    wk.add({
      { "<leader>b", group = "Buffer" },
      { "<leader>c", group = "Code" },
      { "<leader>f", group = "Find" },
      { "<leader>g", group = "Git" },
      { "<leader>h", group = "Hunk" },
      { "<leader>l", group = "LSP" },
      { "<leader>s", group = "Split" },
      { "<leader>t", group = "Terminal" },
    })
  end,
}
```

## Leader Key 設計哲學

好的快捷鍵系統有幾個原則：

### 1. 按功能分組

把相關的操作放在同一個前綴下：

| 前綴 | 功能群組 | 助記 |
|------|---------|------|
| `<leader>f` | **Find** — 搜尋相關 | Find |
| `<leader>g` | **Git** — Git 操作 | Git |
| `<leader>h` | **Hunk** — Git 變更區塊 | Hunk |
| `<leader>l` | **LSP** — 語言伺服器 | LSP |
| `<leader>b` | **Buffer** — Buffer 管理 | Buffer |
| `<leader>s` | **Split** — 視窗分割 | Split |
| `<leader>t` | **Terminal** — Terminal | Terminal |
| `<leader>c` | **Code** — 程式碼操作 | Code |

### 2. 第一個字母盡量和功能名稱對應

```
<leader>ff → Find Files（搜尋檔案）
<leader>fg → Find Grep（搜尋文字）
<leader>fb → Find Buffers（搜尋 buffer）
<leader>gs → Git Status
<leader>gc → Git Commits
```

### 3. 常用操作用更短的快捷鍵

最頻繁使用的操作不需要前綴：

```
<leader>w  → 存檔（太常用了，不需要群組）
<leader>q  → 離開
<leader>e  → 顯示診斷（高頻操作）
```

### 4. 保持一致性

同樣的字母在不同群組中代表類似的意思：

```
<leader>fs → Find Symbols（搜尋符號）
<leader>gs → Git Status（Git 狀態）
<leader>hs → Hunk Stage（Stage 變更）
```

`s` 在不同群組中分別代表 Symbols、Status、Stage——都以 s 開頭。

## 完整的快捷鍵架構

以下是本系列建議的完整快捷鍵映射。所有映射都基於 `<leader>`（空白鍵）：

### 頂層快捷鍵（無群組前綴）

| 快捷鍵 | 功能 | 設定位置 |
|--------|------|---------|
| `<leader>w` | 存檔 | keymaps |
| `<leader>q` | 離開 | keymaps |
| `<leader>e` | 顯示診斷詳情 | LSP |
| `<leader>p` | 貼上不覆蓋 register | keymaps |
| `<leader>d` | 刪除到黑洞 register | keymaps |

### Find 群組 (`<leader>f`)

| 快捷鍵 | 功能 | 來源 |
|--------|------|------|
| `<leader>ff` | 搜尋檔案 | Telescope |
| `<leader>fg` | 全域文字搜尋 | Telescope |
| `<leader>fb` | 搜尋 Buffers | Telescope |
| `<leader>fr` | 最近開啟的檔案 | Telescope |
| `<leader>fw` | 搜尋游標下的文字 | Telescope |
| `<leader>fs` | 搜尋文件符號 | Telescope |
| `<leader>fS` | 搜尋工作區符號 | Telescope |
| `<leader>fh` | 搜尋幫助 | Telescope |
| `<leader>fd` | 搜尋診斷 | Telescope |
| `<leader>fk` | 搜尋快捷鍵 | Telescope |

### Git 群組 (`<leader>g`)

| 快捷鍵 | 功能 | 來源 |
|--------|------|------|
| `<leader>gc` | 搜尋 Git commits | Telescope |
| `<leader>gs` | 搜尋 Git status | Telescope |
| `<leader>gd` | 開啟 Diff View | diffview |
| `<leader>gh` | 檔案 Git 歷史 | diffview |
| `<leader>gH` | 分支 Git 歷史 | diffview |
| `<leader>lg` | 開啟 lazygit | lazygit |

### Hunk 群組 (`<leader>h`)

| 快捷鍵 | 功能 | 來源 |
|--------|------|------|
| `<leader>hs` | Stage hunk | gitsigns |
| `<leader>hr` | Reset hunk | gitsigns |
| `<leader>hS` | Stage buffer | gitsigns |
| `<leader>hR` | Reset buffer | gitsigns |
| `<leader>hu` | Undo stage | gitsigns |
| `<leader>hp` | Preview hunk | gitsigns |
| `<leader>hb` | Blame line | gitsigns |
| `<leader>hd` | Diff this | gitsigns |

### LSP 操作（無前綴，用 `g` 開頭）

| 快捷鍵 | 功能 |
|--------|------|
| `gd` | 跳轉到定義 |
| `gD` | 跳轉到宣告 |
| `gi` | 跳轉到實作 |
| `gr` | 查看引用 |
| `gt` | 跳轉到型別定義 |
| `K` | Hover 文件 |
| `<leader>rn` | 重新命名 |
| `<leader>ca` | 程式碼動作 |
| `<leader>f` | 格式化 |
| `[d` / `]d` | 上/下一個診斷 |

### Split 群組 (`<leader>s`)

| 快捷鍵 | 功能 |
|--------|------|
| `<leader>sv` | 垂直分割 |
| `<leader>sh` | 水平分割 |
| `<leader>sx` | 關閉分割 |

### Terminal 群組 (`<leader>t`)

| 快捷鍵 | 功能 |
|--------|------|
| `<leader>th` | 水平 terminal |
| `<leader>tv` | 垂直 terminal |
| `<leader>tf` | 浮動 terminal |

### Buffer 導航（無前綴）

| 快捷鍵 | 功能 |
|--------|------|
| `<S-h>` | 上一個 buffer |
| `<S-l>` | 下一個 buffer |
| `<leader>bd` | 關閉 buffer |

### 其他導航

| 快捷鍵 | 功能 |
|--------|------|
| `]h` / `[h` | 下/上一個 git hunk |
| `]f` / `[f` | 下/上一個函數 |
| `]c` / `[c` | 下/上一個 class |
| `-` | 開啟檔案瀏覽器 |

## 自訂你的按鍵

以上是建議的預設方案，但快捷鍵是非常個人化的。以下是自訂的原則：

### 找出你的高頻操作

花一天正常開發，記錄你最常做的操作。這些操作應該有最短的快捷鍵。

### 避免衝突

設定新快捷鍵前，用 Telescope 的 `<leader>fk`（search keymaps）檢查是否已被占用。

### 善用 which-key 的群組

當一個前綴下的操作超過 5 個時，考慮是否需要拆分成更小的群組。

### 不要過度自訂

Vim 社群有許多約定俗成的快捷鍵（如 `gd` 跳轉到定義）。盡量遵循這些慣例，這樣：
- 你閱讀別人的配置時能理解
- 別人看你的操作時能理解
- 在全新的 Neovim 環境中也不會迷失

## 練習方法

### 第一週：只用 which-key 提示

不要刻意背快捷鍵。按下 `<leader>` 後看 which-key 的提示，從中選擇需要的操作。

### 第二週：記住群組前綴

目標：不看提示就能按出正確的群組前綴。
- 「我要搜尋」→ `<leader>f`
- 「我要 Git 操作」→ `<leader>g` 或 `<leader>lg`
- 「我要操作 hunk」→ `<leader>h`

### 第三週：記住高頻操作

目標：最常用的 10 個操作不需要看提示。

### 持續：自然地記住

其他操作隨用隨查，不需要刻意記。用多了自然就記住了。

## 小結

快捷鍵系統的設計原則：

1. **按功能分組**（`f` for Find, `g` for Git, etc.）
2. **助記命名**（第一個字母和功能對應）
3. **高頻操作短按鍵**
4. **which-key 提供探索性**

一個好的快捷鍵系統不需要你背任何東西——它的邏輯足夠清楚，讓你「想到功能就能猜到按鍵」。which-key 則在你猜不到時給你提示。

下一篇是系列的最後一篇——我們會做一個完整的總覽，包括從 VSCode 到 Neovim 的功能對照表、常見遷移痛點的解法，以及持續學習的資源。

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
11. **打造快捷鍵系統（本篇）**
12. [從 VSCode 完全畢業](/posts/neovim-series-12-graduation/)
