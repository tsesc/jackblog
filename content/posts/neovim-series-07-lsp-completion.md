+++
title = 'Neovim 系列（七）：程式碼智能——LSP 與自動補全'
date = 2026-03-01T10:00:00+08:00
draft = false
tags = ['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列', 'LSP']
categories = ['技術筆記']
author = 'Jack'
description = '設定 Neovim 的 LSP（Language Server Protocol）和自動補全系統——nvim-lspconfig、mason.nvim、nvim-cmp，讓 Neovim 擁有和 VSCode 一樣的程式碼智能'
toc = true
weight = 7
+++

> 這是 **Neovim 從零開始**系列的第七篇。整個系列共 12 篇文章，將帶你從完全不懂 Vim，到能用 Neovim 打造一個完整的現代化開發環境。

## 什麼是 LSP？

**LSP（Language Server Protocol）** 是微軟在 2016 年為 VSCode 設計的通訊協定，後來成為了業界標準。它的概念很簡單：

- **Language Server**：一個獨立的程式，負責分析特定語言的程式碼
- **Editor**：透過 LSP 協定和 Language Server 溝通

```
┌─────────────┐     LSP      ┌──────────────────┐
│   Neovim    │◄────────────►│  Language Server  │
│  (Editor)   │              │  (如 typescript-  │
│             │              │   language-server) │
└─────────────┘              └──────────────────┘
```

這樣的好處是：每個編輯器只需要實作一次 LSP client，就能支援所有語言。VSCode 的程式碼智能就是基於 LSP，而 Neovim 從 0.5 開始就內建了 LSP client。

### LSP 提供的功能

| 功能 | 說明 | VSCode 對應 |
|------|------|-------------|
| 自動補全 | 輸入時顯示建議 | IntelliSense |
| 跳轉到定義 | 查看函數/變數的定義 | Go to Definition |
| 查看引用 | 找到所有使用某個符號的位置 | Find All References |
| Hover 文件 | 游標停留時顯示型別和文件 | Hover |
| 診斷 | 即時顯示錯誤和警告 | Problems |
| 重新命名 | 全域重新命名符號 | Rename Symbol |
| 程式碼動作 | 快速修復、自動導入 | Quick Fix |
| 格式化 | 自動排版程式碼 | Format Document |

## 我們要安裝的 Plugins

要讓 LSP 和自動補全在 Neovim 中運作，需要幾個 plugin 協同工作：

| Plugin | 用途 |
|--------|------|
| `nvim-lspconfig` | 簡化 LSP server 的設定 |
| `mason.nvim` | 管理 LSP server 的安裝（一鍵安裝） |
| `mason-lspconfig.nvim` | 讓 mason 和 lspconfig 橋接 |
| `nvim-cmp` | 自動補全引擎 |
| `cmp-nvim-lsp` | 從 LSP 獲取補全來源 |
| `cmp-buffer` | 從當前 buffer 內容獲取補全來源 |
| `cmp-path` | 檔案路徑補全 |
| `LuaSnip` | Snippet 引擎 |
| `cmp_luasnip` | Snippet 補全來源 |

看起來很多，但設定起來其實很有系統。

## 設定 Mason + LSP

建立 `lua/plugins/lsp.lua`：

```lua
-- lua/plugins/lsp.lua
return {
  {
    "williamboman/mason.nvim",
    build = ":MasonUpdate",
    config = function()
      require("mason").setup({
        ui = {
          border = "rounded",
          icons = {
            package_installed = "✓",
            package_pending = "➜",
            package_uninstalled = "✗",
          },
        },
      })
    end,
  },
  {
    "williamboman/mason-lspconfig.nvim",
    dependencies = { "williamboman/mason.nvim" },
    config = function()
      require("mason-lspconfig").setup({
        ensure_installed = {
          "lua_ls",           -- Lua
          "ts_ls",            -- TypeScript / JavaScript
          "gopls",            -- Go
          "pyright",          -- Python
          "html",             -- HTML
          "cssls",            -- CSS
          "jsonls",           -- JSON
          "yamlls",           -- YAML
        },
        automatic_installation = true,
      })
    end,
  },
  {
    "neovim/nvim-lspconfig",
    dependencies = {
      "williamboman/mason.nvim",
      "williamboman/mason-lspconfig.nvim",
    },
    config = function()
      local lspconfig = require("lspconfig")
      local capabilities = require("cmp_nvim_lsp").default_capabilities()

      -- 每個 LSP server attach 到 buffer 時設定快捷鍵
      vim.api.nvim_create_autocmd("LspAttach", {
        group = vim.api.nvim_create_augroup("UserLspConfig", {}),
        callback = function(ev)
          local opts = { buffer = ev.buf }

          -- 跳轉
          vim.keymap.set("n", "gd", vim.lsp.buf.definition, opts)
          vim.keymap.set("n", "gD", vim.lsp.buf.declaration, opts)
          vim.keymap.set("n", "gi", vim.lsp.buf.implementation, opts)
          vim.keymap.set("n", "gr", vim.lsp.buf.references, opts)
          vim.keymap.set("n", "gt", vim.lsp.buf.type_definition, opts)

          -- 資訊
          vim.keymap.set("n", "K", vim.lsp.buf.hover, opts)
          vim.keymap.set("n", "<C-k>", vim.lsp.buf.signature_help, opts)

          -- 動作
          vim.keymap.set("n", "<leader>rn", vim.lsp.buf.rename, opts)
          vim.keymap.set({ "n", "v" }, "<leader>ca", vim.lsp.buf.code_action, opts)
          vim.keymap.set("n", "<leader>f", function()
            vim.lsp.buf.format({ async = true })
          end, opts)

          -- 診斷
          vim.keymap.set("n", "[d", vim.diagnostic.goto_prev, opts)
          vim.keymap.set("n", "]d", vim.diagnostic.goto_next, opts)
          vim.keymap.set("n", "<leader>e", vim.diagnostic.open_float, opts)
        end,
      })

      -- 設定各個 LSP server
      local servers = {
        "ts_ls", "gopls", "pyright", "html", "cssls", "jsonls", "yamlls",
      }

      for _, server in ipairs(servers) do
        lspconfig[server].setup({
          capabilities = capabilities,
        })
      end

      -- Lua LSP 需要特殊設定（給 Neovim 開發用）
      lspconfig.lua_ls.setup({
        capabilities = capabilities,
        settings = {
          Lua = {
            diagnostics = {
              globals = { "vim" },  -- 認識 vim 全域變數
            },
            workspace = {
              library = vim.api.nvim_get_runtime_file("", true),
              checkThirdParty = false,
            },
            telemetry = { enable = false },
          },
        },
      })
    end,
  },
}
```

### LSP 快捷鍵總覽

| 快捷鍵 | 功能 | VSCode 對應 |
|--------|------|-------------|
| `gd` | 跳轉到定義 | `F12` 或 `Cmd+Click` |
| `gD` | 跳轉到宣告 | — |
| `gi` | 跳轉到實作 | `Cmd+F12` |
| `gr` | 查看所有引用 | `Shift+F12` |
| `gt` | 跳轉到型別定義 | — |
| `K` | 顯示 hover 文件 | 滑鼠懸停 |
| `<leader>rn` | 重新命名 | `F2` |
| `<leader>ca` | 程式碼動作 | `Cmd+.` |
| `<leader>f` | 格式化 | `Shift+Alt+F` |
| `[d` / `]d` | 上/下一個診斷 | `F8` |
| `<leader>e` | 顯示診斷詳情 | 滑鼠懸停錯誤 |

## 設定自動補全

建立 `lua/plugins/cmp.lua`：

```lua
-- lua/plugins/cmp.lua
return {
  "hrsh7th/nvim-cmp",
  event = "InsertEnter",
  dependencies = {
    "hrsh7th/cmp-nvim-lsp",     -- LSP 補全來源
    "hrsh7th/cmp-buffer",        -- Buffer 補全來源
    "hrsh7th/cmp-path",          -- 路徑補全來源
    "L3MON4D3/LuaSnip",         -- Snippet 引擎
    "saadparwaiz1/cmp_luasnip",  -- Snippet 補全來源
    "rafamadriz/friendly-snippets", -- 常用 snippet 集合
  },
  config = function()
    local cmp = require("cmp")
    local luasnip = require("luasnip")

    -- 載入 VSCode 風格的 snippet
    require("luasnip.loaders.from_vscode").lazy_load()

    cmp.setup({
      snippet = {
        expand = function(args)
          luasnip.lsp_expand(args.body)
        end,
      },
      window = {
        completion = cmp.config.window.bordered(),
        documentation = cmp.config.window.bordered(),
      },
      mapping = cmp.mapping.preset.insert({
        -- 上下選擇
        ["<C-k>"] = cmp.mapping.select_prev_item(),
        ["<C-j>"] = cmp.mapping.select_next_item(),

        -- 滾動文件
        ["<C-b>"] = cmp.mapping.scroll_docs(-4),
        ["<C-f>"] = cmp.mapping.scroll_docs(4),

        -- 觸發補全
        ["<C-Space>"] = cmp.mapping.complete(),

        -- 取消
        ["<C-e>"] = cmp.mapping.abort(),

        -- 確認選擇
        ["<CR>"] = cmp.mapping.confirm({ select = false }),

        -- Tab 鍵行為
        ["<Tab>"] = cmp.mapping(function(fallback)
          if cmp.visible() then
            cmp.select_next_item()
          elseif luasnip.expand_or_jumpable() then
            luasnip.expand_or_jump()
          else
            fallback()
          end
        end, { "i", "s" }),

        ["<S-Tab>"] = cmp.mapping(function(fallback)
          if cmp.visible() then
            cmp.select_prev_item()
          elseif luasnip.jumpable(-1) then
            luasnip.jump(-1)
          else
            fallback()
          end
        end, { "i", "s" }),
      }),

      -- 補全來源（優先順序由上到下）
      sources = cmp.config.sources({
        { name = "nvim_lsp" },   -- LSP
        { name = "luasnip" },    -- Snippets
      }, {
        { name = "buffer" },     -- Buffer 文字
        { name = "path" },       -- 檔案路徑
      }),

      -- 補全選單的圖示
      formatting = {
        format = function(entry, vim_item)
          local source_names = {
            nvim_lsp = "[LSP]",
            luasnip = "[Snip]",
            buffer = "[Buf]",
            path = "[Path]",
          }
          vim_item.menu = source_names[entry.source.name] or ""
          return vim_item
        end,
      },
    })
  end,
}
```

### 補全操作說明

| 快捷鍵 | 功能 |
|--------|------|
| `<C-Space>` | 手動觸發補全 |
| `<C-j>` / `<Tab>` | 選擇下一個補全項 |
| `<C-k>` / `<S-Tab>` | 選擇上一個補全項 |
| `<CR>` | 確認選擇 |
| `<C-e>` | 取消補全 |
| `<C-b>` / `<C-f>` | 滾動文件視窗 |

## 診斷顯示美化

預設的診斷（錯誤、警告）標記比較醜。加入以下設定美化它們：

```lua
-- 可以加在 lsp.lua 的 config 函數開頭
vim.diagnostic.config({
  virtual_text = {
    prefix = "●",
  },
  signs = true,
  underline = true,
  update_in_insert = false,
  severity_sort = true,
  float = {
    border = "rounded",
    source = true,
  },
})

-- 自訂診斷符號
local signs = { Error = " ", Warn = " ", Hint = "󰌵 ", Info = " " }
for type, icon in pairs(signs) do
  local hl = "DiagnosticSign" .. type
  vim.fn.sign_define(hl, { text = icon, texthl = hl, numhl = "" })
end
```

## 使用 Mason 管理 Language Servers

安裝後，按 `:Mason` 開啟管理介面：

- 搜尋你需要的 language server
- 按 `i` 安裝
- 按 `X` 移除
- 按 `u` 更新

因為我們在設定中用了 `ensure_installed`，重啟 Neovim 後 Mason 會自動安裝列表中的 servers。

### 確認 LSP 正常運作

1. 打開一個 TypeScript 或 Go 檔案
2. 執行 `:LspInfo` — 應該顯示對應的 language server 已 attach
3. 把游標放在一個函數上按 `K` — 應該顯示 hover 文件
4. 把游標放在一個函數上按 `gd` — 應該跳轉到定義

## 實戰演練

以一個 TypeScript 檔案為例，體驗 LSP 帶來的能力：

### 1. 自動補全

```typescript
const arr = [1, 2, 3]
arr.  // 輸入 . 後自動顯示 map, filter, reduce 等方法
```

### 2. 跳轉到定義

```typescript
import { useState } from 'react'
//       ^
// 游標放在 useState 上按 gd → 跳到 React 的型別定義
```

### 3. 重新命名

```typescript
function calculateTotal(price: number) {
  //     ^
  // 游標在 calculateTotal 上按 <leader>rn
  // 輸入新名稱 → 所有引用同步更新
}
```

### 4. 程式碼動作

```typescript
// 缺少 import 時
const router = useRouter()  // 紅色波浪線
// 按 <leader>ca → 出現 "Add missing import" 選項
```

## 常見問題排除

### LSP 沒有啟動

```vim
:LspInfo          " 查看 LSP 狀態
:LspLog           " 查看 LSP 日誌
:checkhealth lsp  " 健康檢查
```

### 補全沒有出現

1. 確認 `:LspInfo` 顯示 server 已 attach
2. 確認 `nvim-cmp` 已載入（`:Lazy` 查看）
3. 嘗試 `<C-Space>` 手動觸發

### Language Server 找不到

```vim
:Mason    " 開啟 Mason 檢查安裝狀態
```

確保對應的 language server 已安裝。

## 小結

現在你的 Neovim 已經具備了和 VSCode 同等級的程式碼智能：

- **LSP** 提供定義跳轉、引用查找、hover 文件、診斷、重命名、程式碼動作
- **nvim-cmp** 提供流暢的自動補全，整合 LSP、snippet、buffer、路徑等多個來源
- **Mason** 讓安裝和管理 language servers 變得簡單

下一篇，我們會加入 **Treesitter** 語法高亮和 **Telescope** 模糊搜尋——讓你在程式碼中的瀏覽和搜尋體驗再上一層。
