## ADDED Requirements

### Requirement: Web 框架文章結構
文章 SHALL 使用 Hugo front matter，title 為「Go 語言系列（十三）：Web 框架 — Gin vs Echo」，date 為 2026-02-25T13:00:00+08:00，tags 包含 'Golang系列'，weight = 1。

#### Scenario: Front matter 格式正確
- **WHEN** 文章被 Hugo 讀取
- **THEN** front matter 包含正確的 title、date、tags（含 'Golang系列'）、categories='技術筆記'、author='Jack'、toc=true、weight=1

### Requirement: 為什麼需要 Web 框架
文章 SHALL 說明標準庫 net/http 的局限性，以及 Web 框架在路由、中間件、參數綁定、錯誤處理上提供的價值。

#### Scenario: 標準庫對比
- **WHEN** 讀者閱讀開頭段落
- **THEN** 文章用具體例子展示 net/http 處理路由參數的冗長方式，對比框架的簡潔寫法

### Requirement: Gin 框架介紹
文章 SHALL 介紹 Gin 的核心功能：路由（含參數和群組）、中間件、JSON 綁定與驗證、錯誤處理，MUST 附帶完整可運行的 REST API 範例。

#### Scenario: Gin REST API 範例
- **WHEN** 讀者閱讀 Gin 章節
- **THEN** 文章提供一個包含 CRUD 路由、JSON 綁定、中間件的完整 API 範例

### Requirement: Echo 框架介紹
文章 SHALL 介紹 Echo 的核心功能：路由、中間件、請求綁定、錯誤處理，MUST 用與 Gin 相同的 API 範例來對比。

#### Scenario: Echo REST API 範例
- **WHEN** 讀者閱讀 Echo 章節
- **THEN** 文章提供與 Gin 範例等價的 Echo 實作，方便直接對比

### Requirement: Gin vs Echo 對比表
文章 SHALL 包含一個結構化的對比表，涵蓋效能、API 風格、社群生態、學習曲線等維度。

#### Scenario: 對比總結
- **WHEN** 讀者閱讀對比章節
- **THEN** 文章提供表格對比並給出選擇建議

### Requirement: 系列導覽
文章開頭 SHALL 有引言說明此為系列第十三篇，開始進階篇章。末尾 SHALL 有「下一步」導覽。

#### Scenario: 系列連貫性
- **WHEN** 讀者開啟文章
- **THEN** 開頭有引言段落，末尾引導到下一篇（ORM）
