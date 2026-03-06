## Context

部落格已有一個成功的 Golang 系列（18 篇），使用統一的 front matter 格式、`Golang系列` tag、連續的 weight 值來控制排序。新的 Neovim 系列將沿用相同模式，確保讀者體驗一致。

目標讀者是已有程式開發經驗、目前使用 VSCode 的工程師，想要轉換到 Neovim 並整合 terminal-based 工作流（包含 Claude Code）。

## Goals / Non-Goals

**Goals:**
- 建立完整的 Neovim 學習路徑，從零到可日常使用
- 文章可獨立閱讀，也可循序漸進學習
- 涵蓋現代化 Neovim 生態系統（2024-2025 主流工具）
- 提供從 VSCode 遷移的實戰對照與建議
- 作者本人也能作為學習參考

**Non-Goals:**
- 不涵蓋 Vim script 舊式設定（專注 Lua）
- 不深入特定語言的 LSP 細節設定（只示範通用配置）
- 不建立完整的 dotfiles repo（只在文章中提供片段）
- 不涵蓋 Emacs 或其他編輯器比較

## Decisions

### 1. 文章架構：12 篇漸進式系列

**選擇**：12 篇文章，分為三個階段（基礎 → 設定 → 進階）

**原因**：
- 比 Golang 系列的 10+8 篇稍短，Neovim 主題較集中
- 三階段設計讓讀者可依程度選擇入口
- 12 篇足以涵蓋核心主題，又不會太長讓人望而卻步

**替代方案**：
- 6 篇精簡版：太壓縮，Vim motions 需要充分篇幅
- 20 篇完整版：可以之後再擴充，先求核心完整

### 2. 文章編號與命名

**選擇**：`neovim-series-01-xxx.md` 格式，與 Golang 系列一致

**原因**：保持網站命名一致性，方便管理與排序

### 3. 系列文章規劃

| # | 階段 | 標題方向 | 核心內容 |
|---|------|---------|---------|
| 01 | 基礎 | 為什麼離開 VSCode | 動機、Vim 哲學、modal editing 概念 |
| 02 | 基礎 | Vim 的語言：動詞 + 名詞 | operators, motions, text objects |
| 03 | 基礎 | 移動的藝術 | hjkl、word/paragraph、search、marks |
| 04 | 基礎 | 編輯效率加倍 | registers、macros、dot command、visual mode |
| 05 | 設定 | 安裝 Neovim 與第一個 init.lua | 安裝、基本設定、options、keymaps |
| 06 | 設定 | Plugin 管理：lazy.nvim | plugin manager、安裝流程、常見 plugins |
| 07 | 設定 | 程式碼智能：LSP + 自動補全 | nvim-lspconfig、mason、nvim-cmp |
| 08 | 設定 | 語法高亮與程式碼導航 | Treesitter、Telescope、oil.nvim |
| 09 | 進階 | Git 整合工作流 | lazygit、gitsigns、diffview |
| 10 | 進階 | Terminal 與 Claude Code 整合 | terminal mode、toggleterm、AI 工作流 |
| 11 | 進階 | 打造你的快捷鍵系統 | which-key、leader key 設計、常用映射 |
| 12 | 進階 | 從 VSCode 完全畢業 | 完整配置總覽、常見問題、持續學習資源 |

### 4. 日期與 Weight 策略

**選擇**：
- 日期從 `2026-03-07` 開始，每篇間隔 1 天
- weight 從 1 開始遞增，確保系列內排序正確
- 使用 `+08:00` 時區

### 5. Tag 與分類

**選擇**：
- tags: `['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列']`
- categories: `['技術筆記']`
- 個別文章可追加特定 tag（如 `'LSP'`、`'Git'`）

## Risks / Trade-offs

- **[工具版本更新]** → Neovim 與 plugin 生態更新快速，文章內容可能過時 → 在文章中標註測試版本，並提供官方文件連結
- **[篇幅控制]** → Vim motions 內容龐大，容易寫太長 → 限制每篇 2000-3000 字，用表格和程式碼範例取代大段文字
- **[配置片段分散]** → 讀者可能難以拼湊完整配置 → 第 12 篇提供完整配置總覽
- **[前 4 篇無需安裝 Neovim]** → 讀者可能覺得進度慢 → 在前言說明可先用 VSCode Vim extension 練習
