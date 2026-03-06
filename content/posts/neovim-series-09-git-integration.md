+++
title = 'Neovim 系列（九）：Git 整合工作流——在編輯器中完成所有 Git 操作'
date = 2026-03-15T10:00:00+08:00
draft = false
tags = ['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列', 'Git']
categories = ['技術筆記']
author = 'Jack'
description = '用 gitsigns.nvim 顯示行內變更、lazygit 處理 Git 操作、diffview.nvim 檢視差異——打造完整的 Git 工作流，不需要離開 Neovim'
toc = true
weight = 9
+++

> 這是 **Neovim 從零開始**系列的第九篇。整個系列共 12 篇文章，將帶你從完全不懂 Vim，到能用 Neovim 打造一個完整的現代化開發環境。

## 為什麼在 Neovim 中操作 Git？

VSCode 有內建的 Source Control 面板和 GitLens 擴充，讓你不用離開編輯器就能操作 Git。在 Neovim 中，我們要用幾個 plugins 達到同樣甚至更好的體驗：

- **gitsigns.nvim**：在行號旁顯示 Git 變更狀態
- **lazygit**：功能強大的 Terminal Git UI
- **diffview.nvim**：在 Neovim 中查看 diff

## gitsigns.nvim：行內 Git 狀態

gitsigns 是最基本也最必要的 Git plugin。它在 sign column（行號左邊）顯示每一行的 Git 狀態：

- `│` 綠色：新增的行
- `│` 藍色：修改的行
- `_` 紅色：刪除的行

```lua
-- lua/plugins/gitsigns.lua
return {
  "lewis6991/gitsigns.nvim",
  event = "BufReadPost",
  config = function()
    require("gitsigns").setup({
      signs = {
        add          = { text = "│" },
        change       = { text = "│" },
        delete       = { text = "_" },
        topdelete    = { text = "‾" },
        changedelete = { text = "~" },
      },
      on_attach = function(bufnr)
        local gs = package.loaded.gitsigns
        local opts = { buffer = bufnr }

        -- 導航：在 hunk（變更區塊）之間跳轉
        vim.keymap.set("n", "]h", gs.next_hunk, opts)
        vim.keymap.set("n", "[h", gs.prev_hunk, opts)

        -- Stage / Unstage
        vim.keymap.set("n", "<leader>hs", gs.stage_hunk, opts)
        vim.keymap.set("n", "<leader>hr", gs.reset_hunk, opts)
        vim.keymap.set("v", "<leader>hs", function()
          gs.stage_hunk({ vim.fn.line("."), vim.fn.line("v") })
        end, opts)
        vim.keymap.set("v", "<leader>hr", function()
          gs.reset_hunk({ vim.fn.line("."), vim.fn.line("v") })
        end, opts)

        -- Stage / Reset 整個 buffer
        vim.keymap.set("n", "<leader>hS", gs.stage_buffer, opts)
        vim.keymap.set("n", "<leader>hR", gs.reset_buffer, opts)

        -- Undo stage
        vim.keymap.set("n", "<leader>hu", gs.undo_stage_hunk, opts)

        -- Preview hunk
        vim.keymap.set("n", "<leader>hp", gs.preview_hunk, opts)

        -- Blame
        vim.keymap.set("n", "<leader>hb", function()
          gs.blame_line({ full = true })
        end, opts)
        vim.keymap.set("n", "<leader>tb", gs.toggle_current_line_blame, opts)

        -- Diff
        vim.keymap.set("n", "<leader>hd", gs.diffthis, opts)
      end,
    })
  end,
}
```

### gitsigns 快捷鍵總覽

| 快捷鍵 | 功能 | 說明 |
|--------|------|------|
| `]h` / `[h` | 下/上一個 hunk | 在變更區塊間跳轉 |
| `<leader>hs` | Stage hunk | 暫存當前變更區塊 |
| `<leader>hr` | Reset hunk | 撤銷當前變更區塊 |
| `<leader>hS` | Stage buffer | 暫存整個檔案 |
| `<leader>hR` | Reset buffer | 撤銷整個檔案的變更 |
| `<leader>hu` | Undo stage | 取消暫存 |
| `<leader>hp` | Preview hunk | 預覽變更內容 |
| `<leader>hb` | Blame line | 顯示當前行的 Git blame |
| `<leader>tb` | Toggle blame | 切換行內 blame 顯示 |
| `<leader>hd` | Diff this | 查看當前檔案的 diff |

### 日常使用流程

```
1. 編輯程式碼（sign column 自動顯示變更標記）
2. 用 ]h / [h 在變更區塊間跳轉，檢查修改內容
3. 用 <leader>hp 預覽 hunk，確認要保留的變更
4. 用 <leader>hs stage 個別 hunk（或 <leader>hS stage 整個檔案）
5. 用 <leader>hr 撤銷不需要的變更
```

這種 **hunk 級別**的 staging 比 VSCode 的整個檔案 stage 更精細。你可以只 stage 一個檔案中的部分修改。

## lazygit：終極 Git UI

**lazygit** 是一個獨立的 terminal Git 工具，功能極其強大。它不是 Neovim plugin，而是一個獨立的程式，但和 Neovim 配合得非常好。

### 安裝 lazygit

```bash
brew install lazygit
```

### 在 Neovim 中整合 lazygit

最簡單的方式是用 terminal 開啟：

```lua
-- 加在 keymaps.lua 或另建 lua/plugins/lazygit.lua
vim.keymap.set("n", "<leader>lg", function()
  -- 用浮動視窗開啟 lazygit
  local buf = vim.api.nvim_create_buf(false, true)
  vim.api.nvim_open_win(buf, true, {
    relative = "editor",
    width = math.floor(vim.o.columns * 0.9),
    height = math.floor(vim.o.lines * 0.9),
    col = math.floor(vim.o.columns * 0.05),
    row = math.floor(vim.o.lines * 0.05),
    style = "minimal",
    border = "rounded",
  })
  vim.fn.termopen("lazygit", {
    on_exit = function()
      vim.api.nvim_buf_delete(buf, { force = true })
    end,
  })
  vim.cmd("startinsert")
end, { desc = "Open lazygit" })
```

或者安裝專用的 plugin：

```lua
-- lua/plugins/lazygit.lua
return {
  "kdheepak/lazygit.nvim",
  cmd = "LazyGit",
  dependencies = { "nvim-lua/plenary.nvim" },
  keys = {
    { "<leader>lg", "<cmd>LazyGit<cr>", desc = "Open LazyGit" },
  },
}
```

### lazygit 的常用操作

按 `<leader>lg` 開啟 lazygit 後：

| 面板 | 快捷鍵 | 說明 |
|------|--------|------|
| Files | `Space` | Stage / Unstage 檔案 |
| Files | `a` | Stage / Unstage 所有檔案 |
| Files | `c` | Commit |
| Files | `Enter` | 查看檔案 diff |
| Branches | `Space` | Checkout 分支 |
| Branches | `n` | 新建分支 |
| Branches | `M` | Merge 到目前分支 |
| Commits | `Enter` | 查看 commit 詳情 |
| Commits | `r` | Revert commit |
| 通用 | `p` | Pull |
| 通用 | `P` | Push |
| 通用 | `?` | 顯示幫助 |
| 通用 | `q` | 離開 |

lazygit 的操作非常直覺，幾分鐘就能上手。按 `?` 可以隨時查看幫助。

### 為什麼用 lazygit 而不是 fugitive？

**vim-fugitive** 是 Vim 生態中最經典的 Git plugin，功能強大但學習曲線較陡。**lazygit** 作為一個獨立的 TUI 工具，優勢在於：

- 操作更直覺（類似 GUI）
- 學習曲線更平緩
- 不只 Neovim 可以用，任何 terminal 都可以
- 交互式 rebase 非常方便

## diffview.nvim：視覺化 Diff

diffview 提供了一個專業的 diff 檢視介面，類似 VSCode 的 diff editor：

```lua
-- lua/plugins/diffview.lua
return {
  "sindrets/diffview.nvim",
  cmd = { "DiffviewOpen", "DiffviewFileHistory" },
  keys = {
    { "<leader>gd", "<cmd>DiffviewOpen<cr>", desc = "Open diff view" },
    { "<leader>gh", "<cmd>DiffviewFileHistory %<cr>", desc = "File history" },
    { "<leader>gH", "<cmd>DiffviewFileHistory<cr>", desc = "Branch history" },
  },
  config = function()
    require("diffview").setup({
      enhanced_diff_hl = true,
      view = {
        default = {
          layout = "diff2_horizontal",
        },
      },
    })
  end,
}
```

### diffview 操作

| 快捷鍵 | 功能 |
|--------|------|
| `<leader>gd` | 開啟 diff view（所有未提交的變更） |
| `<leader>gh` | 查看當前檔案的 Git 歷史 |
| `<leader>gH` | 查看整個分支的歷史 |
| `Tab` / `Shift+Tab` | 在檔案間切換 |
| `q` | 關閉 diffview |

## 完整的 Git 工作流

把三個工具組合起來，你可以在 Neovim 中完成完整的 Git 工作流：

### 日常提交流程

```
1. 編輯程式碼
   - sign column 自動顯示哪些行被修改了

2. 檢查變更
   - ]h / [h 跳到變更區塊
   - <leader>hp 預覽每個 hunk 的內容
   - <leader>gd 開啟 diff view 總覽所有變更

3. Staging
   - <leader>hs 逐個 hunk stage（精細控制）
   - 或開啟 lazygit (<leader>lg) 用更直覺的方式 stage

4. Commit
   - 在 lazygit 中按 c 寫 commit message 並提交

5. Push
   - 在 lazygit 中按 P push 到 remote
```

### 查看歷史

```
- <leader>gh  → 查看當前檔案的歷史（誰、何時、改了什麼）
- <leader>hb  → 查看當前行的 blame（這行是誰寫的）
- <leader>tb  → 開啟行內 blame（每行旁邊都顯示作者和時間）
```

### 處理分支

```
- <leader>lg  → 開啟 lazygit
  - 在 Branches 面板中切換、新建、合併分支
  - 在 Commits 面板中 cherry-pick、rebase
```

## 與 VSCode 的 Git 功能對照

| 功能 | VSCode | Neovim |
|------|--------|--------|
| 行內變更標記 | GitLens 或內建 | gitsigns.nvim |
| Stage 檔案 | Source Control 面板 | lazygit |
| Stage 行/hunk | 行內 stage 按鈕 | `<leader>hs` |
| Commit | Source Control 面板 | lazygit |
| Push/Pull | 狀態列按鈕 | lazygit |
| 查看 Diff | Diff Editor | diffview.nvim |
| Git Blame | GitLens inline | `<leader>hb` / `<leader>tb` |
| 檔案歷史 | GitLens File History | `<leader>gh` |
| 分支管理 | 底部狀態列 | lazygit |

## 小結

現在你的 Git 工作流已經完全整合在 Neovim 中了：

| 工具 | 用途 |
|------|------|
| **gitsigns** | 行內變更顯示 + hunk 操作 + blame |
| **lazygit** | 完整的 Git 操作（stage, commit, push, branch） |
| **diffview** | 視覺化 diff + 檔案歷史 |

加上 Telescope 的 Git 相關搜尋（`<leader>gc` 搜尋 commits、`<leader>gs` 查看 git status），你的 Git 體驗是非常完整的。

下一篇，我們要來整合另一個重要的開發工具——**Terminal**，以及探討如何在 Neovim 中高效使用 **Claude Code**。
