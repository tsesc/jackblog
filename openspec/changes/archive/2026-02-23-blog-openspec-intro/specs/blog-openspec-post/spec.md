## ADDED Requirements

### Requirement: 文章 Front Matter 完整配置
文章 SHALL 使用 Hugo 標準 front matter，包含 title、date、tags、categories、author、description、toc 等欄位。categories 設為「技術筆記」。

#### Scenario: Front matter 格式正確
- **WHEN** 文章被 Hugo 解析
- **THEN** 所有 front matter 欄位正確填入，文章可正常渲染於部落格首頁與文章列表

### Requirement: 文章包含 OpenSpec 核心概念介紹
文章 SHALL 清楚說明 OpenSpec 的定位（Spec-Driven Development 框架）、解決的問題（AI 輔助開發缺乏結構化需求溝通）、以及核心理念（fluid not rigid、iterative not waterfall 等）。

#### Scenario: 讀者理解 OpenSpec 定位
- **WHEN** 讀者閱讀完核心概念章節
- **THEN** 讀者能理解 OpenSpec 是什麼、為何需要它、以及它的設計哲學

### Requirement: 文章包含工作流程說明
文章 SHALL 介紹 OpenSpec 的完整工作流程：new → ff/continue → apply → verify → archive，並說明每個階段的用途。

#### Scenario: 讀者理解操作流程
- **WHEN** 讀者閱讀工作流程章節
- **THEN** 讀者能理解從建立 change 到歸檔的完整流程

### Requirement: 文章包含使用情境與實際範例
文章 SHALL 提供至少兩個具體使用情境（如：個人專案快速開發、團隊協作功能開發），並包含 CLI 指令範例。

#### Scenario: 讀者能對照自身需求
- **WHEN** 讀者閱讀使用情境章節
- **THEN** 讀者能判斷 OpenSpec 是否適合自己的開發場景

### Requirement: 文章包含與其他工具的比較
文章 SHALL 簡要比較 OpenSpec 與其他類似方案（如 GitHub Spec Kit、AWS Kiro、純 prompt 開發）的差異。

#### Scenario: 讀者理解差異化優勢
- **WHEN** 讀者閱讀比較章節
- **THEN** 讀者能理解 OpenSpec 相較於其他方案的核心優勢

### Requirement: 文章以繁體中文撰寫
文章 SHALL 全文使用繁體中文，技術專有名詞（如 Spec-Driven Development、delta spec）保留英文並適當說明。

#### Scenario: 語言一致性
- **WHEN** 文章發布
- **THEN** 全文為繁體中文，技術名詞以英文標示並附帶解釋
