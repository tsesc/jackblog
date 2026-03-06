## Why

長期使用 VSCode 的開發者想要轉換到 Neovim，主要動機有二：一是手不想頻繁離開鍵盤去碰滑鼠，提升效率；二是搭配 Claude Code 等 terminal-based 工具時，希望整個開發流程都在 terminal 完成。目前部落格已有完整的 Golang 系列作為參考模板，需要一套同樣完整、從零開始的 Neovim 教學系列。這系列文章也是作者本人的學習筆記。

## What Changes

- 新增 10+ 篇 Neovim 系列教學文章（`neovim-series-01` ~ `neovim-series-XX`）
- 文章涵蓋範圍：
  - Vim 基本操作與 modal editing 概念
  - Vim motions、text objects、operators 完整教學
  - 從 VSCode 遷移的心態與實戰建議
  - Neovim 安裝、設定與基礎配置（init.lua）
  - 現代化 plugin manager（lazy.nvim）
  - LSP、自動補全、語法高亮（Treesitter）
  - 檔案導航（Telescope、oil.nvim）
  - Git 整合（lazygit、gitsigns）
  - Terminal 整合與 Claude Code 搭配工作流
  - 進階技巧：macros、registers、自訂快捷鍵
  - 打造完整開發環境的最終配置
- 所有文章使用與 Golang 系列一致的 front matter 格式
- 使用 `Neovim系列` tag 進行系列文章歸類

## Capabilities

### New Capabilities
- `neovim-blog-series`: 完整的 Neovim 從零開始教學系列文章集合，包含基礎 Vim 操作、Neovim 現代化設定、plugin 生態系統、開發工作流整合等主題

### Modified Capabilities

（無需修改現有功能）

## Impact

- `content/posts/`: 新增 10+ 篇 markdown 檔案（neovim-series-*.md）
- 不影響現有文章與網站設定
- 不需修改 hugo.toml 或 theme 設定
