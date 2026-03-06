+++
title = 'Neovim 系列（四）：編輯效率加倍——Registers、Macros 與 Dot Command'
date = 2026-02-26T10:00:00+08:00
draft = false
tags = ['Neovim', 'Vim', '編輯器', '開發工具', 'Neovim系列']
categories = ['技術筆記']
author = 'Jack'
description = '學習 Vim 的進階編輯技巧：registers 剪貼簿系統、macros 錄製巨集、dot command 重複指令、visual mode 選取操作，讓你的編輯效率倍增'
toc = true
weight = 4
+++

> 這是 **Neovim 從零開始**系列的第四篇。整個系列共 12 篇文章，將帶你從完全不懂 Vim，到能用 Neovim 打造一個完整的現代化開發環境。

## 效率的下一個層次

前兩篇我們學了 Vim 的語言系統和移動指令。有了這些基礎，你已經能在程式碼中自由移動和編輯了。

這篇要教的三個功能——**Dot Command**、**Registers** 和 **Macros**——會把你的效率再推上一個層次。它們的核心概念是：**重複操作的自動化**。

## Dot Command：最強的單一按鍵

`.`（dot）這個按鍵可能是 Vim 中投資報酬率最高的功能。它的作用很簡單：

> **重複上一次的修改操作**

### 基本用法

```javascript
// 想把所有的 var 改成 const
var name = "Jack"
var age = 30
var city = "Taipei"
```

操作步驟：
1. 游標在第一行的 `var` 上
2. 按 `cw` 然後輸入 `const`，按 `Esc`
3. 移到下一行的 `var` 上，按 `.`——自動執行「把單字改成 const」
4. 再移到下一行，按 `.`——同樣的操作

**一個 `.` 就重複了整個 `cw` + 輸入 + `Esc` 的序列**。

### Dot Command 的威力

`.` 重複的是上一次「從 Normal mode 進入 Insert mode 到離開」的完整操作，或者是上一次 Normal mode 下的操作指令。

| 上一次操作 | `.` 會做什麼 |
|-----------|------------|
| `ciwfoo<Esc>` | 把游標下的單字改成 foo |
| `dd` | 刪除一整行 |
| `A;<Esc>` | 在行尾加上分號 |
| `I// <Esc>` | 在行首加上 `// ` |
| `>>` | 向右縮排一次 |

### 設計你的操作讓 `.` 更好用

好的 Vim 使用者會**刻意設計操作方式**，讓後續能用 `.` 重複。

```javascript
// 差的做法：用 / 搜尋 + cw 修改
// 好的做法：

// 想在多個行尾加分號
A;<Esc>     // 在行尾加分號
j.          // 下一行，重複
j.          // 再下一行，重複

// 想刪除多行
dd          // 刪除一行
.           // 再刪一行
.           // 再刪一行
```

## Registers：Vim 的多重剪貼簿

在 VSCode 中，剪貼簿只有一個。但 Vim 有多個 **registers**——你可以把不同的內容存在不同的 register 中。

### 查看 Registers

```vim
:registers    " 或 :reg
```

### Register 種類

| Register | 名稱 | 用途 |
|----------|------|------|
| `""` | 未命名 register | 最近一次 `d`、`c`、`y` 的內容（預設） |
| `"0` | Yank register | 最近一次 `y` 的內容 |
| `"1` ~ `"9` | 數字 register | 最近 9 次刪除的內容（歷史） |
| `"a` ~ `"z` | 命名 register | 你主動存入的內容 |
| `"+` | 系統剪貼簿 | 和 OS 共用的剪貼簿 |
| `"*` | 選取剪貼簿 | X11 的 primary selection |
| `"_` | 黑洞 register | 丟進去的東西消失（不影響其他 register） |
| `".` | 上次插入 | 上一次在 Insert mode 輸入的文字 |
| `"/` | 搜尋 register | 上一次搜尋的 pattern |

### 使用方式

在任何 `d`、`c`、`y`、`p` 操作前加上 `"{register}`：

```
"ayy      → 複製整行到 register a
"ap       → 貼上 register a 的內容
"+yy      → 複製整行到系統剪貼簿
"+p       → 從系統剪貼簿貼上
```

### 最常遇到的問題：刪除覆蓋了複製

這可能是 Vim 新手最大的困擾：

```
1. yy 複製一行
2. dd 刪除另一行
3. p 貼上 → 結果貼出的是剛刪除的行，不是複製的那行！
```

原因：`dd` 的內容覆蓋了未命名 register `""`。

**解決方法（任選一種）**：

1. **用 `"0p`**：register `0` 永遠保存最後一次 yank 的內容
2. **用命名 register**：`"ayy` 存到 a，`"ap` 從 a 取出
3. **用黑洞 register 刪除**：`"_dd` 刪除但不存入任何 register

### 實用搭配

```
"ayiw     → 複製游標下的單字到 register a
"byi{     → 複製大括號內容到 register b
...移動到其他地方...
"ap       → 貼上 register a 的單字
"bp       → 貼上 register b 的大括號內容
```

## Macros：錄製你的操作

如果 Dot Command 是「重複上一步」，那 Macros 就是「重複一整個操作流程」。

### 基本操作

| 指令 | 功能 |
|------|------|
| `q{register}` | 開始錄製到指定 register（例如 `qa`） |
| `q` | 停止錄製 |
| `@{register}` | 播放 register 中的 macro（例如 `@a`） |
| `@@` | 重播上一次播放的 macro |
| `{n}@{register}` | 播放 N 次 |

### 錄製 Macro 的步驟

```
qa        → 開始錄製到 register a（狀態列會顯示 recording @a）
...操作... → 執行你想重複的操作
q         → 停止錄製
@a        → 播放
10@a      → 播放 10 次
```

### 實戰範例 1：批次加引號

假設有一串沒有引號的文字：

```
apple
banana
cherry
grape
```

想把每行都加上引號變成 `"apple",`：

1. 游標在第一行開頭
2. `qa` — 開始錄製
3. `I"` — 在行首插入引號
4. `<Esc>A",` — 在行尾加上引號和逗號
5. `<Esc>j0` — 回到 Normal mode，往下一行，到行首
6. `q` — 停止錄製
7. `3@a` — 對剩下的 3 行執行

結果：
```
"apple",
"banana",
"cherry",
"grape",
```

### 實戰範例 2：重構函數呼叫

```javascript
// 把這些呼叫改成 await
fetchUser(id)
fetchPosts(userId)
fetchComments(postId)
```

1. `qa` — 開始錄製
2. `Iawait ` — 在行首加 await
3. `<Esc>j0` — 下一行行首
4. `q` — 停止錄製
5. `2@a` — 執行 2 次

### Macro 錄製的技巧

**讓 Macro 可靠的關鍵**：使用**結構化移動**而非相對移動。

```
// 差：用 3l 移動 3 格（如果下一行長度不同就出錯）
// 好：用 w 移到下一個單字（不管間距多少）
// 好：用 0 移到行首（不管游標在哪）
// 好：用 f( 找到括號（不管位置在哪）
```

**Macro 開始和結束時的游標位置要一致**，這樣才能連續播放。通常在結尾加上 `j0`（下一行行首）。

## Visual Mode：視覺化選取

Visual Mode 讓你先選取文字，再決定要做什麼操作。這和一般編輯器的「先選取再操作」很像，但 Vim 的選取方式更強大。

### 三種 Visual Mode

| 指令 | 模式 | 用途 |
|------|------|------|
| `v` | Character visual | 字元級選取 |
| `V` | Line visual | 整行選取 |
| `Ctrl-v` | Block visual | 矩形區塊選取 |

### Character Visual (`v`)

```javascript
// 從游標開始選取到某個位置
v$        → 選取到行尾
viw       → 選取整個單字
vi"       → 選取引號內容
v%        → 選取到配對括號
```

選取後可以：
- `d` — 刪除
- `c` — 修改
- `y` — 複製
- `>` — 縮排
- `u` — 轉小寫
- `U` — 轉大寫
- `:` — 對選取範圍執行命令

### Line Visual (`V`)

```
V         → 選取整行
Vj        → 選取兩行
V}        → 選取到段落結尾
Vip       → 選取整個段落
```

最常見的用法：`Vjjjd`（選取多行後刪除）或 `Vjjj>`（選取多行後縮排）。

### Block Visual (`Ctrl-v`)：最酷的功能

Block Visual 是 Vim 的獨門功能——矩形區塊選取。

```javascript
// 想在多行前面加上 //
const a = 1
const b = 2
const c = 3
const d = 4
```

操作步驟：
1. 游標在第一行開頭
2. `Ctrl-v` — 進入 Block Visual
3. `3j` — 往下選取 4 行
4. `I// ` — 輸入 `// `
5. `Esc` — 完成，4 行都加上了 `// `

```javascript
// const a = 1
// const b = 2
// const c = 3
// const d = 4
```

**Block Visual 的其他用法**：
- `Ctrl-v` + 選取 + `d` → 刪除矩形區塊
- `Ctrl-v` + 選取 + `A` + 文字 + `Esc` → 在每行尾部追加文字
- `Ctrl-v` + 選取 + `c` + 文字 + `Esc` → 替換矩形區塊

## Undo / Redo

| 指令 | 功能 |
|------|------|
| `u` | Undo（復原） |
| `Ctrl-r` | Redo（重做） |

Vim 的 undo 是基於樹狀結構（undo tree），而不是線性的。這意味著你不會因為 undo 後做了新操作就丟失歷史。Neovim 還有插件可以視覺化這棵 undo tree。

## 貼上（Paste）的技巧

| 指令 | 功能 |
|------|------|
| `p` | 在游標**後面**貼上 |
| `P` | 在游標**前面**貼上 |
| `gp` | 貼上後游標移到貼上內容的末尾 |
| `gP` | 同上但在游標前貼上 |

### 整行貼上 vs 行內貼上

`p` 的行為取決於被複製內容的類型：

- 如果是用 `yy` 複製的（line-wise），`p` 會貼在下一行
- 如果是用 `yw` 複製的（character-wise），`p` 會貼在游標後

## 替換指令

| 指令 | 功能 |
|------|------|
| `r{char}` | 替換游標下的字元為 `{char}` |
| `R` | 進入 Replace mode（持續覆蓋文字） |
| `:s/old/new/g` | 替換當前行的所有 old 為 new |
| `:%s/old/new/g` | 替換整個檔案的所有 old 為 new |
| `:%s/old/new/gc` | 同上但每次替換前確認 |

### 替換指令的範圍

```vim
:s/foo/bar/g          " 只替換當前行
:%s/foo/bar/g         " 替換整個檔案
:5,10s/foo/bar/g      " 替換第 5 到 10 行
:'<,'>s/foo/bar/g     " 替換 Visual 選取的範圍
```

## 綜合練習

### 練習 1：Dot Command
1. 打開一個有多行 `console.log(...)` 的檔案
2. 在第一行的 `console` 前面加上 `// `（用 `I// <Esc>`）
3. 用 `j.` 重複對下面每一行做同樣的操作

### 練習 2：Registers
1. `"ayy` 複製第一行到 register a
2. `"byy` 複製第二行到 register b
3. 移到檔案其他位置
4. `"ap` 和 `"bp` 分別貼上

### 練習 3：Macros
1. 建立一個每行有一個單字的清單
2. 錄製一個 macro，把每行包裝成 `<li>word</li>`
3. 用 `@a` 或 `10@a` 對所有行執行

### 練習 4：Block Visual
1. 找一段多行程式碼
2. 用 `Ctrl-v` 選取多行的開頭
3. 用 `I` 在每行前面加上相同的文字

## 小結

本篇學到的功能讓你可以處理**批次操作**和**重複性工作**：

| 功能 | 用途 | 適合場景 |
|------|------|---------|
| `.` | 重複上一步 | 簡單的重複修改 |
| Registers | 多重剪貼簿 | 同時保存多段內容 |
| Macros | 錄製操作序列 | 複雜的批次操作 |
| Block Visual | 矩形選取 | 多行同時編輯 |

到這裡，你已經掌握了 Vim 操作的所有核心基礎。這四篇的內容，即使你一直用 VSCode + Vim extension，也完全適用。

下一篇開始，我們要正式進入 Neovim 的世界——安裝 Neovim、寫你的第一個 `init.lua` 配置檔。

---

## Neovim 系列文章導航

**基礎篇**
1. [為什麼我想離開 VSCode？](/posts/neovim-series-01-why-leave-vscode/)
2. [Vim 的語言——動詞、名詞、組合技](/posts/neovim-series-02-vim-language/)
3. [移動的藝術](/posts/neovim-series-03-movement/)
4. **編輯效率加倍（本篇）**

**設定篇**
5. [安裝 Neovim 與 init.lua](/posts/neovim-series-05-install-and-init-lua/)
6. [Plugin 管理 lazy.nvim](/posts/neovim-series-06-plugin-management/)
7. [程式碼智能 LSP + 補全](/posts/neovim-series-07-lsp-completion/)
8. [語法高亮與檔案導航](/posts/neovim-series-08-treesitter-telescope/)

**進階篇**
9. [Git 整合工作流](/posts/neovim-series-09-git-integration/)
10. [Terminal 與 Claude Code 整合](/posts/neovim-series-10-terminal-claude-code/)
11. [打造快捷鍵系統](/posts/neovim-series-11-keymap-system/)
12. [從 VSCode 完全畢業](/posts/neovim-series-12-graduation/)
