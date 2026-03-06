+++
title = 'Neovim 系列（八）：語法高亮與檔案導航——Treesitter + Telescope'
date = 2026-03-02T10:00:00+08:00
draft = false
tags = ['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列', 'Treesitter', 'Telescope']
categories = ['技術筆記']
author = 'Jack'
description = '用 Treesitter 實現精確的語法高亮，用 Telescope 取代 VSCode 的 Cmd+P 和 Cmd+Shift+F，再加上 oil.nvim 檔案瀏覽——打造完整的程式碼導航體驗'
toc = true
weight = 8
+++

> 這是 **Neovim 從零開始**系列的第八篇。整個系列共 12 篇文章，將帶你從完全不懂 Vim，到能用 Neovim 打造一個完整的現代化開發環境。

## 本篇的目標

經過前幾篇的設定，你的 Neovim 已經有了 LSP 和自動補全。這篇要加入三個讓開發體驗再度提升的功能：

1. **Treesitter**：基於語法樹的精確高亮和文字操作
2. **Telescope**：強大的模糊搜尋（取代 VSCode 的 Cmd+P）
3. **oil.nvim**：檔案瀏覽器（取代 VSCode 的左側邊欄）

## Treesitter：真正理解你的程式碼

### 傳統語法高亮 vs Treesitter

傳統的語法高亮用**正則表達式**來匹配——它只看文字模式，不懂程式碼結構。這導致：

- 複雜的語法可能標錯顏色
- 巢狀結構容易混亂
- 不同語言嵌套（如 HTML 中的 JavaScript）效果差

**Treesitter** 完全不同——它**解析程式碼為語法樹**（AST），真正理解程式碼的結構。好處是：

- 語法高亮更精確（區分函數名、變數、型別、常數等）
- 支援增量解析（編輯時只重新解析修改的部分，非常快）
- 提供基於語法結構的文字操作（如選取整個函數）

### 安裝 Treesitter

```lua
-- lua/plugins/treesitter.lua
return {
  "nvim-treesitter/nvim-treesitter",
  build = ":TSUpdate",
  event = "BufReadPost",
  dependencies = {
    "nvim-treesitter/nvim-treesitter-textobjects",
  },
  config = function()
    require("nvim-treesitter.configs").setup({
      ensure_installed = {
        "lua", "vim", "vimdoc",
        "javascript", "typescript", "tsx",
        "go", "gomod", "gosum",
        "python",
        "html", "css", "json", "yaml", "toml",
        "markdown", "markdown_inline",
        "bash", "dockerfile",
        "gitcommit", "diff",
      },
      auto_install = true,
      highlight = {
        enable = true,
      },
      indent = {
        enable = true,
      },
      incremental_selection = {
        enable = true,
        keymaps = {
          init_selection = "<C-space>",
          node_incremental = "<C-space>",
          scope_incremental = false,
          node_decremental = "<bs>",
        },
      },
    })
  end,
}
```

### 增量選取

上面設定中的 `incremental_selection` 是一個非常好用的功能：

1. 按 `Ctrl+Space` 開始選取（選取目前的語法節點）
2. 再按 `Ctrl+Space` 擴大選取（選取父節點）
3. 按 `Backspace` 縮小選取

```javascript
// 游標在 "hello" 上
const msg = "hello world"

// Ctrl+Space → 選取 "hello world"（字串內容）
// Ctrl+Space → 選取 "hello world"（含引號）
// Ctrl+Space → 選取 msg = "hello world"（賦值表達式）
// Ctrl+Space → 選取整行（宣告語句）
```

這比手動用 Visual mode 精確多了。

### Treesitter Text Objects

安裝 `nvim-treesitter-textobjects` 後，你可以用語法結構來操作：

```lua
-- 加在 treesitter.lua 的 config 中
textobjects = {
  select = {
    enable = true,
    lookahead = true,
    keymaps = {
      ["af"] = "@function.outer",    -- 選取整個函數
      ["if"] = "@function.inner",    -- 選取函數內容
      ["ac"] = "@class.outer",       -- 選取整個 class
      ["ic"] = "@class.inner",       -- 選取 class 內容
      ["aa"] = "@parameter.outer",   -- 選取參數（含逗號）
      ["ia"] = "@parameter.inner",   -- 選取參數（不含逗號）
    },
  },
  move = {
    enable = true,
    set_jumps = true,
    goto_next_start = {
      ["]f"] = "@function.outer",    -- 跳到下一個函數
      ["]c"] = "@class.outer",       -- 跳到下一個 class
    },
    goto_previous_start = {
      ["[f"] = "@function.outer",    -- 跳到上一個函數
      ["[c"] = "@class.outer",       -- 跳到上一個 class
    },
  },
},
```

現在你可以：

| 指令 | 功能 |
|------|------|
| `daf` | 刪除整個函數 |
| `vif` | 選取函數內容 |
| `yaf` | 複製整個函數 |
| `cia` | 修改一個參數 |
| `]f` | 跳到下一個函數 |
| `[f` | 跳到上一個函數 |

這些操作**理解程式碼結構**，不再只是文字模式匹配。

## Telescope：萬能模糊搜尋

Telescope 是 Neovim 生態中最重要的 plugin 之一。它提供了一個統一的模糊搜尋介面，可以搜尋幾乎任何東西：檔案、文字、Git 記錄、LSP 符號等。

### 前置需求

Telescope 需要幾個外部工具來發揮最佳效能：

```bash
# ripgrep — 極快的文字搜尋
brew install ripgrep

# fd — 極快的檔案搜尋
brew install fd
```

### 安裝 Telescope

```lua
-- lua/plugins/telescope.lua
return {
  "nvim-telescope/telescope.nvim",
  branch = "0.1.x",
  dependencies = {
    "nvim-lua/plenary.nvim",
    {
      "nvim-telescope/telescope-fzf-native.nvim",
      build = "make",
    },
  },
  config = function()
    local telescope = require("telescope")
    local actions = require("telescope.actions")

    telescope.setup({
      defaults = {
        path_display = { "truncate" },
        sorting_strategy = "ascending",
        layout_config = {
          horizontal = {
            prompt_position = "top",
            preview_width = 0.55,
          },
          width = 0.87,
          height = 0.80,
        },
        mappings = {
          i = {
            ["<C-j>"] = actions.move_selection_next,
            ["<C-k>"] = actions.move_selection_previous,
            ["<C-q>"] = actions.send_selected_to_qflist + actions.open_qflist,
            ["<Esc>"] = actions.close,
          },
        },
        file_ignore_patterns = {
          "node_modules",
          ".git/",
          "dist/",
          "build/",
          "%.lock",
        },
      },
      pickers = {
        find_files = {
          hidden = true,
        },
      },
    })

    -- 載入 fzf extension（加快搜尋速度）
    telescope.load_extension("fzf")

    -- 快捷鍵
    local builtin = require("telescope.builtin")

    -- 檔案搜尋
    vim.keymap.set("n", "<leader>ff", builtin.find_files, { desc = "Find files" })
    vim.keymap.set("n", "<leader>fr", builtin.oldfiles, { desc = "Recent files" })
    vim.keymap.set("n", "<leader>fb", builtin.buffers, { desc = "Find buffers" })

    -- 文字搜尋
    vim.keymap.set("n", "<leader>fg", builtin.live_grep, { desc = "Live grep" })
    vim.keymap.set("n", "<leader>fw", builtin.grep_string, { desc = "Grep word under cursor" })

    -- Git
    vim.keymap.set("n", "<leader>gc", builtin.git_commits, { desc = "Git commits" })
    vim.keymap.set("n", "<leader>gs", builtin.git_status, { desc = "Git status" })

    -- LSP
    vim.keymap.set("n", "<leader>fs", builtin.lsp_document_symbols, { desc = "Document symbols" })
    vim.keymap.set("n", "<leader>fS", builtin.lsp_workspace_symbols, { desc = "Workspace symbols" })

    -- 其他
    vim.keymap.set("n", "<leader>fh", builtin.help_tags, { desc = "Help tags" })
    vim.keymap.set("n", "<leader>fd", builtin.diagnostics, { desc = "Diagnostics" })
    vim.keymap.set("n", "<leader>fk", builtin.keymaps, { desc = "Keymaps" })
  end,
}
```

### Telescope 快捷鍵 vs VSCode 對照

| Telescope | 功能 | VSCode 對應 |
|-----------|------|-------------|
| `<leader>ff` | 搜尋檔案 | `Cmd+P` |
| `<leader>fg` | 全域文字搜尋 | `Cmd+Shift+F` |
| `<leader>fb` | 切換 Buffer | `Cmd+Tab` |
| `<leader>fr` | 最近開啟的檔案 | `Cmd+P` 歷史 |
| `<leader>fw` | 搜尋游標下的文字 | 選取文字 → `Cmd+Shift+F` |
| `<leader>fs` | 搜尋文件中的符號 | `Cmd+Shift+O` |
| `<leader>fh` | 搜尋幫助文件 | — |

### Telescope 內的操作

| 快捷鍵 | 功能 |
|--------|------|
| `<C-j>` / `<C-k>` | 上下移動選擇 |
| `<CR>` | 開啟選中項目 |
| `<C-x>` | 水平分割開啟 |
| `<C-v>` | 垂直分割開啟 |
| `<C-q>` | 發送到 quickfix list |
| `<Esc>` | 關閉 |

### 使用技巧

**模糊搜尋語法**（在搜尋框中）：

| 語法 | 功能 |
|------|------|
| `foo` | 模糊匹配 foo |
| `'exact` | 精確匹配 exact |
| `^prefix` | 前綴匹配 |
| `suffix$` | 後綴匹配 |
| `!exclude` | 排除包含 exclude 的結果 |

例如：搜尋 `'spec .ts$` 會找到所有精確包含 "spec" 且以 ".ts" 結尾的檔案。

## oil.nvim：檔案瀏覽器

在 VSCode 中你習慣左側的檔案樹。在 Neovim 中，**oil.nvim** 提供了一個更 Vim-native 的方案——它把目錄當成一個可編輯的 buffer。你可以用 Vim 的所有操作來管理檔案。

```lua
-- lua/plugins/oil.lua
return {
  "stevearc/oil.nvim",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  config = function()
    require("oil").setup({
      default_file_explorer = true,
      columns = {
        "icon",
      },
      view_options = {
        show_hidden = true,
      },
      keymaps = {
        ["g?"] = "actions.show_help",
        ["<CR>"] = "actions.select",
        ["<C-v>"] = "actions.select_vsplit",
        ["<C-s>"] = "actions.select_split",
        ["-"] = "actions.parent",
        ["_"] = "actions.open_cwd",
        ["`"] = "actions.cd",
        ["g."] = "actions.toggle_hidden",
      },
    })

    vim.keymap.set("n", "-", "<CMD>Oil<CR>", { desc = "Open file explorer" })
  end,
}
```

### oil.nvim 的操作

| 操作 | 功能 |
|------|------|
| `-` | 打開檔案瀏覽器（或返回上層目錄） |
| `<CR>` | 進入目錄 / 開啟檔案 |
| 在 buffer 中新增一行文字 | 建立新檔案 |
| 在 buffer 中刪除一行 (`dd`) | 刪除檔案 |
| 在 buffer 中修改檔名 | 重新命名檔案 |
| `:w` | 確認所有變更 |

是的，你沒看錯——**用 Vim 的編輯操作來管理檔案**。新增一行就是新增檔案，刪除一行就是刪除檔案。這就是 Vim 哲學的精髓。

### 另一個選擇：neo-tree

如果你比較習慣 VSCode 風格的側邊欄檔案樹，可以用 **neo-tree**：

```lua
-- lua/plugins/neo-tree.lua（替代方案）
return {
  "nvim-neo-tree/neo-tree.nvim",
  branch = "v3.x",
  dependencies = {
    "nvim-lua/plenary.nvim",
    "nvim-tree/nvim-web-devicons",
    "MunifTanjim/nui.nvim",
  },
  config = function()
    require("neo-tree").setup({
      close_if_last_window = true,
      filesystem = {
        follow_current_file = { enabled = true },
        filtered_items = {
          visible = true,
          hide_dotfiles = false,
        },
      },
    })
    vim.keymap.set("n", "<leader>e", ":Neotree toggle<CR>", { desc = "Toggle file explorer" })
  end,
}
```

我個人推薦 **oil.nvim**——它更符合 Vim 的思維方式。但如果你剛從 VSCode 轉過來，neo-tree 的側邊欄可能更親切。

## 功能對照總表

以下整理本篇設定的功能與 VSCode 的對照：

| 功能 | VSCode | Neovim |
|------|--------|--------|
| 語法高亮 | 內建 | Treesitter |
| 搜尋檔案 | `Cmd+P` | `<leader>ff` (Telescope) |
| 全域搜尋 | `Cmd+Shift+F` | `<leader>fg` (Telescope) |
| 搜尋符號 | `Cmd+Shift+O` | `<leader>fs` (Telescope) |
| 切換檔案 | `Cmd+Tab` | `<leader>fb` (Telescope) |
| 檔案瀏覽器 | 側邊欄 | `-` (oil.nvim) |
| 選取函數 | 手動選取 | `vaf` (Treesitter) |
| 跳到下個函數 | — | `]f` (Treesitter) |

## 驗證安裝

重啟 Neovim 後，確認以下幾點：

1. **Treesitter**：打開一個程式碼檔案，語法高亮應該更精確（變數、函數、型別有不同顏色）
2. **Telescope**：按 `<leader>ff`，應該出現模糊搜尋檔案的介面
3. **Telescope grep**：按 `<leader>fg`，輸入文字應該能全域搜尋
4. **oil.nvim**：按 `-`，應該看到當前目錄的檔案列表

如果 Treesitter 的高亮不正確，執行 `:TSUpdate` 更新所有語法解析器。

## 小結

到這裡，設定篇的四篇文章就完成了。你的 Neovim 已經具備：

| 功能 | 提供者 |
|------|--------|
| 基本配置 | init.lua (options + keymaps) |
| Plugin 管理 | lazy.nvim |
| 漂亮外觀 | catppuccin + lualine |
| 程式碼智能 | LSP + nvim-cmp |
| 精確語法高亮 | Treesitter |
| 模糊搜尋 | Telescope |
| 檔案瀏覽 | oil.nvim |

這已經是一個非常強大的開發環境了！接下來的進階篇會加入更多工作流整合：Git、Terminal、快捷鍵系統。

下一篇，我們來整合 **Git 工作流**——讓你在 Neovim 中完成所有 Git 操作。

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
8. **語法高亮與檔案導航（本篇）**

**進階篇**
9. [Git 整合工作流](/posts/neovim-series-09-git-integration/)
10. [Terminal 與 Claude Code 整合](/posts/neovim-series-10-terminal-claude-code/)
11. [打造快捷鍵系統](/posts/neovim-series-11-keymap-system/)
12. [從 VSCode 完全畢業](/posts/neovim-series-12-graduation/)
