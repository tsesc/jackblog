+++
title = 'Claude Code Skills 完整指南：打造專屬 AI 工作流程'
date = 2025-12-03T10:00:00+08:00
draft = false
tags = ['AI工具', 'Claude', 'Claude Code', 'Skills', '開發工具', 'Anthropic']
categories = ['技術筆記']
author = 'Jack'
description = '深入了解 Claude Code Skills 功能，學習如何創建、配置和使用 Skills 來擴展 Claude 的能力，提升開發效率'
toc = true
+++

## 什麼是 Claude Code Skills？

Claude Code Skills 是 Anthropic 於 2025 年 10 月推出的模組化能力擴展功能。它允許你將專業知識、工作流程和腳本打包成可重複使用的「技能包」，讓 Claude 能夠根據上下文自動識別並啟用這些能力。

### 核心特點

- **組合性（Composable）**：多個 Skills 可以自動協同工作
- **可攜性（Portable）**：同一格式適用於 Claude.ai、Claude Code 和 API
- **高效性（Efficient）**：採用漸進式載入，最小化 Token 消耗
- **強大性（Powerful）**：可包含可執行程式碼，實現確定性任務執行

### Skills vs Slash Commands vs MCP

| 比較項目 | Skills | Slash Commands | MCP Tools |
|---------|--------|----------------|-----------|
| **觸發方式** | 模型自動識別 | 使用者手動輸入 `/command` | 使用者或模型觸發 |
| **複雜度** | 支援多檔案結構 | 單一 Markdown 檔案 | 需要獨立服務程序 |
| **發現機制** | 根據上下文自動啟用 | 需要明確調用 | 啟動時載入 |
| **Token 效率** | 初始僅數十個 Token | 調用時載入 | 通常 10,000+ Token |

Skills 提供了一種比 MCP 更簡單的替代方案——透過 CLI 工具和 Markdown 文件，取代複雜的協議規範。

## Skills 目錄結構

### 存放位置

Skills 可以存放在三個位置：

```bash
# 個人 Skills（跨專案通用）
~/.claude/skills/skill-name/SKILL.md

# 專案 Skills（團隊共享，可版本控制）
.claude/skills/skill-name/SKILL.md

# 插件 Skills（隨插件安裝）
<plugin-root>/skills/skill-name/SKILL.md
```

### 完整目錄結構

```
skill-name/
├── SKILL.md           # 必需 - 入口檔案
├── reference.md       # 可選 - 詳細文件
├── examples.md        # 可選 - 使用範例
├── scripts/           # 可選 - 可執行腳本
│   ├── helper.py
│   └── validator.py
├── templates/         # 可選 - 文件模板
│   └── report.md
└── data/              # 可選 - 參考資料
    └── config.json
```

## SKILL.md 檔案格式

SKILL.md 是 Skill 的核心入口檔案，由 YAML frontmatter 和 Markdown 內容組成：

```yaml
---
name: skill-name-here
description: 這個 Skill 的功能描述，以及何時應該使用它。包含特定的觸發關鍵字。
allowed-tools: Read, Grep, Glob  # 可選 - 限制可用工具
license: MIT                      # 可選 - 授權資訊
metadata:                         # 可選 - 自定義屬性
  author: your-name
  version: 1.0.0
---

# Skill 標題

## 概述
簡要說明這個 Skill 的功能。

## 前置條件
- 必要的相依套件
- 系統需求

## 使用說明
1. 工作流程第一步
2. 工作流程第二步
3. 工作流程第三步

## 範例
### 範例 1：基本用法
[詳細範例]

## 錯誤處理
- 常見錯誤的處理方式
- 備援程序

## 限制
- 已知的限制條件
- 超出範圍的請求
```

### 必填欄位

| 欄位 | 要求 | 最大長度 |
|------|------|----------|
| `name` | 只能使用小寫字母、數字和連字號，必須與目錄名稱相符 | 64 字元 |
| `description` | 說明功能和使用時機，這對自動發現至關重要 | 1024 字元 |

### 選填欄位

| 欄位 | 用途 |
|------|------|
| `allowed-tools` | 逗號分隔的工具清單，限制 Claude 只能使用指定工具 |
| `license` | 授權資訊或授權檔案的參考 |
| `metadata` | 自定義的鍵值對屬性 |

## 如何創建 Skills

### 步驟一：定義問題

識別一個具體、可重複且有可衡量結果的任務。好的候選任務：

- 已經做過至少 5 次，未來還會重複 10 次以上的任務
- 需要特定領域知識的工作流程
- 可從確定性程式碼執行中受益的流程

### 步驟二：創建目錄結構

```bash
# 個人 Skills
mkdir -p ~/.claude/skills/my-skill-name

# 專案 Skills（推薦用於團隊共享）
mkdir -p .claude/skills/my-skill-name
```

### 步驟三：命名規範

- 使用小寫字母和連字號：`pdf-editor`、`brand-guidelines`、`code-reviewer`
- 保持名稱簡短、清晰且具描述性
- 避免使用 `utils` 或 `helper` 等通用術語

### 步驟四：撰寫描述

描述是決定 Claude 何時啟用 Skill 的關鍵：

```yaml
# 不好的描述（避免）
description: 處理 PDF

# 好的描述（推薦）
description: 從 PDF 檔案擷取文字和表格、填寫表單、合併文件。當處理 PDF 檔案或使用者提到 PDF、表單或文件擷取時使用。
```

描述的最佳實踐：

- 使用第三人稱視角
- 包含觸發關鍵字
- 結合「做什麼」和「何時使用」
- 指定檔案類型或使用者可能提到的技術術語

### 步驟五：撰寫主要說明

使用清晰的層級結構：

- 概述部分
- 前置條件
- 逐步執行說明
- 具體範例
- 錯誤處理指南
- 限制說明

## 實際範例

### 範例一：Git Commit 訊息生成器

這是一個最小化的單檔案 Skill：

**檔案**：`.claude/skills/commit-message-generator/SKILL.md`

```yaml
---
name: commit-message-generator
description: 從暫存的變更生成清晰、符合規範的 commit 訊息。當撰寫 git commits、檢視 diffs 或準備提交程式碼時使用。
---

# Commit 訊息生成器

## 使用說明
1. 執行 `git diff --staged` 查看待提交的變更
2. 分析變更類型（feature、fix、refactor 等）
3. 按照 Conventional Commits 格式生成 commit 訊息：
   - Type：feat、fix、refactor、docs、test、chore
   - Scope：可選，受影響的組件
   - Subject：祈使語氣，最多 50 字元
   - Body：解釋什麼和為什麼，每行 72 字元

## 格式
```
<type>(<scope>): <subject>

<body>

<footer>
```

## 範例
### 新增功能
```
feat(auth): 新增 JWT Token 更新端點

實作自動 Token 更新以防止會話逾時。
Token 會在過期前 5 分鐘自動更新。

Closes #123
```
```

### 範例二：唯讀程式碼審查器

使用 `allowed-tools` 限制只能讀取檔案：

**檔案**：`.claude/skills/code-reviewer/SKILL.md`

```yaml
---
name: code-reviewer
description: 審查程式碼的最佳實踐、安全漏洞和效能問題。當分析程式碼品質、審查 PR 或稽核現有程式碼時使用。
allowed-tools: Read, Grep, Glob
---

# 程式碼審查器

## 審查清單

### 1. 程式碼組織
- 每個函式/類別單一職責
- 適當的抽象層級
- 清晰的命名慣例

### 2. 錯誤處理
- 適當的例外處理
- 有意義的錯誤訊息
- 快速失敗原則

### 3. 效能
- 高效的演算法
- 資源管理
- 快取機會

### 4. 安全性
- 輸入驗證
- SQL 注入防護
- 敏感資料處理

### 5. 測試覆蓋率
- 單元測試存在
- 邊界案例覆蓋
- 測試品質

## 輸出格式
按嚴重程度組織發現：
- **Critical**：安全漏洞、資料遺失風險
- **High**：效能問題、缺少錯誤處理
- **Medium**：程式碼風格、可維護性問題
- **Low**：小改進、建議
```

### 範例三：PDF 處理器（含腳本）

包含多個支援檔案的完整 Skill：

**目錄結構**：
```
.claude/skills/pdf-processor/
├── SKILL.md
├── reference.md
└── scripts/
    ├── extract_text.py
    ├── fill_form.py
    └── merge_pdfs.py
```

**檔案**：`.claude/skills/pdf-processor/SKILL.md`

```yaml
---
name: pdf-processor
description: 從 PDF 檔案擷取文字和表格、填寫表單、合併文件。當處理 PDF 檔案或使用者提到 PDF、表單或文件擷取時使用。
---

# PDF 處理器

## 前置條件
需要的 Python 套件：
- pypdf
- pdfplumber
- reportlab

安裝：`pip install pypdf pdfplumber reportlab`

## 功能

### 文字擷取
從 PDF 檔案擷取文字內容：
```bash
python scripts/extract_text.py input.pdf
```

### 表單填寫
使用提供的資料填寫 PDF 表單：
```bash
python scripts/fill_form.py template.pdf data.json output.pdf
```

### 文件合併
將多個 PDF 合併為一個：
```bash
python scripts/merge_pdfs.py output.pdf file1.pdf file2.pdf file3.pdf
```

## 工作流程
1. 識別需要的 PDF 操作
2. 檢查輸入檔案是否存在且可存取
3. 執行適當的腳本
4. 在回傳給使用者前驗證輸出

詳細 API 文件請參閱 [reference.md](reference.md)。
```

### 範例四：品牌設計指南

用於設計相關工作的 Skill：

```yaml
---
name: brand-guidelines
description: 將公司品牌顏色、字型和設計標準應用於文件和產出物。當創建品牌內容、簡報或設計資產時使用。
---

# 品牌設計指南

## 色彩系統

### 主要顏色
- **黑色**：#141413（文字、標題）
- **珊瑚色**：#d97757（強調、CTA）
- **沙色**：#f5f0e8（背景）

### 次要顏色
- **深灰**：#3d3d3c（次要文字）
- **淺珊瑚**：#e8a88a（hover 狀態）

## 字型規範

### 字體
- **標題**：Poppins（600 字重）
- **內文**：Lora（400 字重）
- **程式碼**：JetBrains Mono

### 尺寸
- H1：36px / 2.25rem
- H2：28px / 1.75rem
- H3：22px / 1.375rem
- 內文：16px / 1rem
- 小字：14px / 0.875rem

## 使用準則
1. 主要元素始終使用品牌顏色
2. 保持一致的字型層級
3. 謹慎使用珊瑚色作為強調
4. 確保足夠的對比度以符合無障礙標準
```

### 範例五：Excel 分析器

處理試算表的 Skill：

```yaml
---
name: excel-analyzer
description: 分析 Excel 試算表、建立樞紐分析表並生成圖表。當處理 Excel 檔案、試算表或分析 .xlsx 格式的表格資料時使用。
---

# Excel 分析器

## 功能
- 讀取和解析 .xlsx 檔案
- 建立摘要統計
- 生成樞紐分析表
- 建立圖表和視覺化
- 套用專業格式

## 工作流程
1. 使用 openpyxl 或 pandas 載入 Excel 檔案
2. 分析資料結構和類型
3. 執行請求的分析
4. 生成格式化的輸出或視覺化
5. 儲存結果到新的工作表或檔案

## 常用操作

### 摘要統計
```python
import pandas as pd
df = pd.read_excel('data.xlsx')
print(df.describe())
```

### 樞紐分析表
```python
pivot = pd.pivot_table(df,
    values='sales',
    index='region',
    columns='quarter',
    aggfunc='sum')
```
```

## Skills 的自動載入機制

Skills 採用三層漸進式載入系統：

1. **第一層（元資料）**：啟動時僅載入 YAML frontmatter 中的 `name` 和 `description`（數十個 Token）
2. **第二層（核心內容）**：當 Claude 判斷相關時，載入完整的 SKILL.md 內容
3. **第三層+（補充資料）**：根據需要載入額外的參考檔案

這種設計確保了高效的 Token 使用，同時保持完整的功能存取。

## 查看可用的 Skills

### 直接詢問 Claude

```
有哪些可用的 Skills？
列出所有可用的 Skills
顯示這個專案中的 Skills
```

### 檢查檔案系統

```bash
ls ~/.claude/skills/           # 個人 Skills
ls .claude/skills/             # 專案 Skills
cat .claude/skills/my-skill/SKILL.md
```

## 最佳實踐

### 何時使用 Skills

| 使用 Skills 當... | 使用 Slash Commands 當... | 使用 MCP 當... |
|------------------|--------------------------|---------------|
| Claude 應該自動識別任務 | 你想要明確控制觸發時機 | 需要即時外部 API 存取 |
| 需要多個檔案/腳本 | 說明適合單一檔案 | 需要即時資料擷取 |
| 涉及複雜驗證步驟 | 快速、常用的提示 | 需要認證流程 |
| 團隊需要標準化工作流程 | 個人捷徑 | 需要雙向通訊 |

### 描述撰寫指南

1. **使用第三人稱**：「處理 Excel 檔案」而非「我處理 Excel 檔案」
2. **包含觸發關鍵字**：檔案類型、技術術語、動作動詞
3. **結合做什麼和何時使用**：「從 PDF 擷取文字。當處理 PDF 檔案時使用。」
4. **具體明確**：避免模糊的術語如「處理文件」
5. **最多 1024 字元**：簡潔但全面

### 檔案組織

1. **SKILL.md 保持在 500 行以內**以優化 Token 使用
2. **參考檔案只深入一層**：從 SKILL.md 直接連結，避免巢狀引用
3. **使用基於領域的組織**：按主題區分不同檔案
4. **超過 100 行的檔案包含目錄**
5. **使用 Unix 風格路徑**：始終使用正斜線（`scripts/helper.py`）

### 撰寫有效的 Skill 提示

1. **從真實用例開始**：為你經常重複的任務建立 Skills
2. **定義成功標準**：指定所需的輸出格式和品質門檻
3. **假設 Claude 很聰明**：避免過度解釋明顯的概念
4. **選擇單一術語**：在整個文件中一致使用術語
5. **呈現單一方法**：提供清晰的主要路徑，並為替代方案提供逃生艙
6. **包含驗證迴圈**：「執行驗證器、修復錯誤、重複」的模式
7. **提供實用腳本**：預先編寫的腳本比生成的程式碼更可靠

## 測試與除錯

### 測試方法

```bash
# 啟用除錯模式
claude --debug
```

提出應該觸發你的 Skill 的問題，觀察 Claude 是否啟用它。

### 常見問題與解決方案

| 問題 | 解決方案 |
|------|----------|
| Claude 忽略 Skill | 讓描述更具體，加入觸發關鍵字 |
| Skill 不載入 | 驗證路徑、檢查 YAML 語法、確保 `---` 分隔符正確 |
| 錯誤的 Skill 被啟用 | 使用不同的觸發術語來區分 Skills |
| 檔案只被部分讀取 | 保持檔案聚焦，避免深層巢狀引用 |

## 安全考量

- 只從可信來源安裝 Skills
- 使用前審核捆綁的腳本和程式碼相依套件
- 檢視網路連線和外部 API 呼叫
- 適當時使用 `allowed-tools` 限制功能

## 專案 Skills vs 個人 Skills

| 面向 | 專案 Skills（`.claude/skills/`） | 個人 Skills（`~/.claude/skills/`） |
|------|--------------------------------|----------------------------------|
| **範圍** | 單一專案 | 所有專案 |
| **版本控制** | 提交到 git | 不受版本控制 |
| **團隊共享** | 自動與團隊成員共享 | 需要手動共享 |
| **使用案例** | 專案特定工作流程 | 個人工具 |

## 官方資源

### 主要文件

- [Claude Code Skills 文件](https://docs.anthropic.com/en/docs/claude-code/skills)
- [Skills 部落格公告](https://www.anthropic.com/news/agent-skills)
- [最佳實踐指南](https://docs.anthropic.com/en/docs/build-with-claude/agent-skills/best-practices)

### 官方儲存庫

- **Anthropic Skills Repository**：[github.com/anthropics/skills](https://github.com/anthropics/skills)
  - 創意、開發、企業和文件任務的官方範例 Skills
  - 建立新 Skills 的模板
  - Agent Skills 規範（v1.0，2025 年 10 月 16 日發布）

- **Claude Office Skills**：[github.com/anthropics/claude-office-skills](https://github.com/anthropics/claude-office-skills)
  - PowerPoint (PPTX)、Word (DOCX)、Excel (XLSX)、PDF 功能
  - Claude 桌面版使用的相同 Skills

## 總結

Claude Code Skills 代表了透過簡單的檔案導向方法擴展 AI 能力的重大進展。主要優點：

1. **簡單性**：只需一個包含 SKILL.md 檔案的資料夾即可打包複雜的專業知識
2. **效率**：漸進式載入只載入必要的內容，節省 Token
3. **自主性**：Claude 根據上下文決定何時使用 Skills，無需明確調用
4. **可攜性**：相同格式適用於 Claude.ai、Claude Code 和 API
5. **團隊友善**：專案 Skills 受版本控制並自動共享

Skills 是讓 AI 代理更有能力同時保持實作對非程式設計師也容易上手的關鍵功能。透過 Markdown 和可選的腳本，任何人都可以開始建立自己的專屬 AI 工作流程。

---

*本文基於 Claude Code 官方文件及 Anthropic 發布的 Agent Skills 規範撰寫。*
