## ADDED Requirements

### Requirement: Series consists of 12 progressive articles
The blog SHALL contain exactly 12 articles in the Neovim series, numbered `neovim-series-01` through `neovim-series-12`, covering three stages: 基礎 (01-04), 設定 (05-08), 進階 (09-12).

#### Scenario: All 12 articles exist
- **WHEN** the content/posts/ directory is listed
- **THEN** files `neovim-series-01-*.md` through `neovim-series-12-*.md` SHALL all exist

### Requirement: Each article uses consistent front matter
Each article SHALL use TOML front matter with: title, date, draft=false, tags (including 'Neovim系列'), categories=['技術筆記'], author='Jack', description, toc=true, and weight matching the article number.

#### Scenario: Front matter validation
- **WHEN** any neovim-series article is opened
- **THEN** it SHALL contain all required front matter fields
- **THEN** tags SHALL include 'Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列'
- **THEN** weight SHALL equal the article number (01→1, 02→2, etc.)

### Requirement: Articles follow sequential date ordering
Articles SHALL be dated starting from 2026-03-07, with each subsequent article dated one day later, using +08:00 timezone.

#### Scenario: Date sequence validation
- **WHEN** article dates are compared
- **THEN** neovim-series-01 SHALL have date 2026-03-07T10:00:00+08:00
- **THEN** each subsequent article SHALL increment the date by exactly one day

### Requirement: Article 01 covers motivation and Vim philosophy
Article 01 SHALL explain why a VSCode user would switch to Neovim, introduce modal editing concept, and cover the Vim philosophy of composable commands.

#### Scenario: Reader understands the motivation
- **WHEN** a VSCode user reads article 01
- **THEN** they SHALL understand the benefits of modal editing
- **THEN** they SHALL understand the keyboard-centric workflow advantage
- **THEN** the article SHALL mention compatibility with terminal tools like Claude Code

### Requirement: Article 02 covers Vim's verb-noun language
Article 02 SHALL teach operators (d, c, y, etc.), motions (w, b, e, etc.), and text objects (iw, i", a(, etc.) as a composable language system.

#### Scenario: Reader learns operator + motion pattern
- **WHEN** a reader completes article 02
- **THEN** they SHALL understand how to combine operators with motions (e.g., `dw`, `ci"`, `ya(`)
- **THEN** the article SHALL include a reference table of common operators, motions, and text objects

### Requirement: Article 03 covers advanced movement
Article 03 SHALL cover all major movement commands: hjkl, word-level (w/b/e/W/B/E), line-level (0/^/$), paragraph ({ }), search (//?/n/N), f/t character search, and marks.

#### Scenario: Reader can navigate efficiently
- **WHEN** a reader completes article 03
- **THEN** they SHALL know at least 15 distinct movement commands
- **THEN** the article SHALL include practice exercises

### Requirement: Article 04 covers editing power features
Article 04 SHALL cover registers, macros (q), dot command (.), visual mode (v/V/Ctrl-V), and undo/redo.

#### Scenario: Reader can use advanced editing
- **WHEN** a reader completes article 04
- **THEN** they SHALL understand how to record and replay macros
- **THEN** they SHALL understand named registers and the unnamed register
- **THEN** they SHALL understand block visual mode for column editing

### Requirement: Article 05 covers Neovim installation and init.lua
Article 05 SHALL guide installation on macOS (Homebrew), introduce the Lua configuration system, and set up basic options and keymaps in init.lua.

#### Scenario: Reader has working Neovim setup
- **WHEN** a reader follows article 05
- **THEN** they SHALL have Neovim installed and running
- **THEN** they SHALL have a basic init.lua with common options (line numbers, tabs, clipboard, etc.)
- **THEN** they SHALL have leader key and basic keymaps configured

### Requirement: Article 06 covers plugin management with lazy.nvim
Article 06 SHALL introduce lazy.nvim as the plugin manager, explain the plugin spec format, and install essential UI plugins (colorscheme, statusline, icons).

#### Scenario: Reader has plugin system working
- **WHEN** a reader follows article 06
- **THEN** they SHALL have lazy.nvim installed and configured
- **THEN** they SHALL have at least one colorscheme and statusline plugin installed
- **THEN** the article SHALL explain lazy-loading concepts

### Requirement: Article 07 covers LSP and auto-completion
Article 07 SHALL set up nvim-lspconfig, mason.nvim for LSP server management, and nvim-cmp for auto-completion with multiple sources.

#### Scenario: Reader has code intelligence working
- **WHEN** a reader follows article 07
- **THEN** they SHALL have LSP providing diagnostics, go-to-definition, and hover documentation
- **THEN** they SHALL have auto-completion working with LSP and snippet sources
- **THEN** the article SHALL demonstrate with at least one language (TypeScript or Go)

### Requirement: Article 08 covers syntax highlighting and file navigation
Article 08 SHALL configure Treesitter for syntax highlighting, Telescope for fuzzy finding, and oil.nvim or neo-tree for file browsing.

#### Scenario: Reader can navigate codebase
- **WHEN** a reader follows article 08
- **THEN** they SHALL have Treesitter providing accurate syntax highlighting
- **THEN** they SHALL be able to fuzzy-find files, grep content, and browse directories
- **THEN** the article SHALL compare these to VSCode equivalents (Cmd+P, Cmd+Shift+F, sidebar)

### Requirement: Article 09 covers Git integration
Article 09 SHALL integrate Git tools: gitsigns.nvim for inline git status, lazygit or fugitive for Git operations, and diffview.nvim for diff viewing.

#### Scenario: Reader can do Git workflow in Neovim
- **WHEN** a reader follows article 09
- **THEN** they SHALL see git changes inline in the editor
- **THEN** they SHALL be able to stage, commit, and view diffs without leaving Neovim

### Requirement: Article 10 covers terminal and Claude Code integration
Article 10 SHALL explain Neovim's built-in terminal mode, toggleterm.nvim for terminal management, and practical workflow combining Neovim with Claude Code in the terminal.

#### Scenario: Reader has integrated terminal workflow
- **WHEN** a reader follows article 10
- **THEN** they SHALL understand terminal mode keybindings (Ctrl-\ Ctrl-N to exit)
- **THEN** they SHALL have toggleterm configured for quick terminal access
- **THEN** the article SHALL show a practical Claude Code + Neovim workflow

### Requirement: Article 11 covers keymap system design
Article 11 SHALL introduce which-key.nvim for discoverability, leader key philosophy, and guide readers to design their own keybinding system organized by function groups.

#### Scenario: Reader has organized keymaps
- **WHEN** a reader follows article 11
- **THEN** they SHALL have which-key showing available keybindings
- **THEN** they SHALL understand how to organize keymaps by prefix (e.g., `<leader>f` for find, `<leader>g` for git)

### Requirement: Article 12 covers complete configuration and graduation from VSCode
Article 12 SHALL provide a complete configuration overview, address common migration pain points, list useful resources for continued learning, and give a VSCode-to-Neovim feature mapping table.

#### Scenario: Reader has complete working environment
- **WHEN** a reader completes article 12
- **THEN** they SHALL have a feature mapping table (VSCode feature → Neovim equivalent)
- **THEN** they SHALL have a list of resources for continued learning
- **THEN** the article SHALL address top 10 common pain points when switching from VSCode

### Requirement: Each article includes series navigation
Each article SHALL include an opening note identifying it as part of the Neovim series with total article count, similar to the Golang series format.

#### Scenario: Series identification
- **WHEN** any neovim-series article is opened
- **THEN** the first line after front matter SHALL be a blockquote introducing the series
- **THEN** it SHALL state the current article number and total count

### Requirement: Articles are written in Traditional Chinese
All article content SHALL be written in Traditional Chinese (繁體中文). Technical terms, commands, and code SHALL remain in English.

#### Scenario: Language consistency
- **WHEN** any article is read
- **THEN** prose content SHALL be in Traditional Chinese
- **THEN** code blocks, command names, and plugin names SHALL be in English
