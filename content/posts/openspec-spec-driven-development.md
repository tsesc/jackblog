+++
title = 'OpenSpec：讓 AI 先讀懂需求再寫程式的 Spec-Driven Development 框架'
date = 2026-02-24T10:00:00+08:00
draft = false
tags = ['AI工具', 'OpenSpec', 'Spec-Driven Development', '開發流程', 'Claude Code', '開源工具']
categories = ['技術筆記']
author = 'Jack'
description = '介紹 OpenSpec 這個開源的 Spec-Driven Development 框架，如何在 AI 輔助開發中建立結構化的需求溝通機制，提升程式碼產出的可預測性'
toc = true
+++

## AI 寫程式很快，但方向對嗎？

如果你有在用 Claude Code、Cursor、GitHub Copilot 這類 AI coding assistant，應該都有過類似的經驗：跟 AI 描述完需求，它刷刷刷寫了一堆程式碼，結果一看——方向完全歪掉。

問題不在 AI 不夠聰明，而是我們跟它溝通的方式太隨意。需求散落在對話歷史裡，沒有結構、沒有共識，AI 自然容易猜錯。尤其在修改既有系統時，這個問題更加嚴重——AI 不知道現在系統長什麼樣，也不知道你想改哪裡。

**OpenSpec** 就是為了解決這個問題而生的。它在人類意圖和 AI 實作之間加了一層輕量的規格（spec）層，讓你和 AI 在寫程式之前，先對「要做什麼」達成共識。

## 什麼是 Spec-Driven Development？

Spec-Driven Development（SDD）的核心概念很簡單：**先寫規格，再寫程式**。

這不是什麼新概念——傳統軟體工程的 PRD、RFC、Design Doc 都是類似的思路。但 OpenSpec 把這件事做得特別適合 AI 時代的開發節奏：

- **Fluid not rigid**：沒有死板的階段閘門，想改隨時改
- **Iterative not waterfall**：邊做邊學，逐步完善
- **Easy not complex**：設定簡單，儀式感低
- **Built for brownfield**：專為修改既有系統設計，不只是 greenfield 專案

OpenSpec 是 [Fission AI](https://github.com/Fission-AI/OpenSpec) 開發的開源框架，以 MIT 授權釋出，GitHub 上已累積超過 25,000 顆星。

## OpenSpec 的核心概念

### 1. Change — 每個改動都是一個獨立單元

當你要做一個功能或修一個 bug，就建立一個 change。每個 change 是一個資料夾，裡面包含四種文件（artifact）：

| Artifact | 用途 |
|----------|------|
| `proposal.md` | 為什麼要做？做什麼？ |
| `specs/*.md` | 具體的行為規格（Given/When/Then） |
| `design.md` | 技術方案怎麼做？ |
| `tasks.md` | 實作清單，可勾選追蹤進度 |

### 2. Delta Spec — 只描述差異

這是 OpenSpec 最巧妙的設計之一。你不需要重寫整份規格，只需要描述「變了什麼」：

- **ADDED**：新增的需求
- **MODIFIED**：修改的行為（必須包含完整更新內容）
- **REMOVED**：移除的功能（附理由與遷移方案）

這個設計讓 OpenSpec 特別適合 brownfield 開發——你的專案不需要從零開始寫完所有規格，只要描述這次改了什麼就好。

### 3. Archive — 規格隨系統一起成長

當一個 change 完成後，delta spec 會合併回主規格（`openspec/specs/`），change 本身則歸檔保留。這意味著你的規格是「活的」——它隨著系統一起演進，每次改動都有完整的歷史記錄。

## 實際工作流程

OpenSpec 的工作流程可以用五個動作串起來：

```
/opsx:new → /opsx:ff → /opsx:apply → /opsx:verify → /opsx:archive
```

### Step 1：建立 Change

```bash
/opsx:new add-dark-mode
```

在 `openspec/changes/add-dark-mode/` 下建立腳手架。

### Step 2：產生 Artifacts

你可以一步到位：

```bash
/opsx:ff    # Fast-forward，一次產生所有 artifacts
```

或是逐步建立，邊做邊調整：

```bash
/opsx:continue    # 每次建立下一個 artifact
```

AI 會根據依賴順序（proposal → specs + design → tasks）依序產生文件。

### Step 3：實作

```bash
/opsx:apply
```

AI 根據 tasks.md 的清單逐一實作，完成一項就打勾。規格和設計文件提供了足夠的脈絡，讓 AI 不會偏離方向。

### Step 4：驗證

```bash
/opsx:verify
```

從三個維度檢查實作是否符合規格：完整性（Completeness）、正確性（Correctness）、一致性（Coherence）。

### Step 5：歸檔

```bash
/opsx:archive
```

Delta spec 合併回主規格，change 資料夾移到歸檔目錄。一次迭代完成。

## 使用情境

### 情境一：個人專案快速開發

你有一個 side project，想加上使用者認證功能。以前你可能直接跟 AI 說「幫我加 JWT 認證」，然後花大量時間來回修正 AI 的產出。

使用 OpenSpec：

1. `/opsx:new add-auth` — 建立 change
2. `/opsx:ff` — AI 會先問清楚：要用 JWT 還是 session？Token 存哪裡？哪些 endpoint 需要保護？然後產生完整的 proposal、spec、design、tasks
3. `/opsx:apply` — 按照雙方同意的規格實作
4. `/opsx:archive` — 歸檔，規格留存

整個過程可能只比直接 prompt 多花十分鐘，但省下的來回修正時間遠超過這個投入。

### 情境二：團隊協作功能開發

你的團隊要同時開發支付系統和通知系統。兩個功能可以各開一個 change，平行推進：

```
openspec/changes/
├── add-payment/          # 支付系統
│   ├── proposal.md
│   ├── specs/
│   ├── design.md
│   └── tasks.md
└── add-notifications/    # 通知系統
    ├── proposal.md
    ├── specs/
    ├── design.md
    └── tasks.md
```

每個 change 的規格獨立管理，完成後各自歸檔合併。如果兩邊的 spec 有衝突，歸檔時會提示處理——就像 git merge conflict 的概念。

## 與其他方案的比較

| 特性 | OpenSpec | GitHub Spec Kit | AWS Kiro | 直接 Prompt |
|------|----------|-----------------|----------|-------------|
| 工作流彈性 | 自由迭代 | 嚴格階段閘門 | IDE 綁定 | 無結構 |
| 工具支援 | 24+ AI 工具 | 主要 GitHub 生態 | 僅 Kiro IDE | 任意 |
| 開源授權 | MIT | 開源 | 封閉 | N/A |
| Brownfield 支援 | Delta Spec 設計 | 有限 | 有限 | 無 |
| 設定成本 | 低（npm install） | 中等 | 低（但被鎖定） | 零 |
| 產出可預測性 | 高 | 高 | 中 | 低 |

如果你已經有慣用的 AI 工具（Claude Code、Cursor 等），OpenSpec 的最大優勢是**不需要換工具**——它作為一層 spec layer 疊加在你現有的工作流程上。

## 快速開始

安裝只需要兩步：

```bash
npm install -g @fission-ai/openspec@latest
cd your-project && openspec init
```

`openspec init` 會互動式地問你使用哪些 AI 工具，然後自動安裝對應的 skill 檔案。不需要 API key，完全在本地運行。

## 結語

AI 輔助開發的瓶頸已經不是 AI 會不會寫程式，而是我們能不能精準地告訴它要寫什麼。OpenSpec 提供了一個務實的解決方案：不是增加繁瑣的流程，而是在你和 AI 之間建立一份「契約」，讓雙方在動手之前先達成共識。

如果你的日常開發已經重度依賴 AI coding assistant，而且經常覺得「AI 寫的東西不太對但說不上來哪裡不對」，不妨試試 OpenSpec。花一點時間先把需求理清楚，換來的是更可預測、更少返工的開發體驗。

**相關連結：**
- [GitHub Repository](https://github.com/Fission-AI/OpenSpec)
- [官方文件](https://github.com/Fission-AI/OpenSpec/tree/main/docs)
- [Discord 社群](https://discord.gg/YctCnvvshC)
