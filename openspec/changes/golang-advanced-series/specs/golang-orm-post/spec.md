## ADDED Requirements

### Requirement: ORM 文章結構
文章 SHALL 使用 Hugo front matter，title 為「Go 語言系列（十四）：ORM 與資料庫工具 — GORM vs sqlx」，date 為 2026-02-25T14:00:00+08:00，tags 包含 'Golang系列'，weight = 1。

#### Scenario: Front matter 格式正確
- **WHEN** 文章被 Hugo 讀取
- **THEN** front matter 包含正確的 title、date、tags（含 'Golang系列'）、categories='技術筆記'、author='Jack'、toc=true、weight=1

### Requirement: GORM 介紹
文章 SHALL 介紹 GORM 的核心功能：Model 定義（struct tag）、AutoMigrate、CRUD 操作、關聯（HasOne/HasMany/BelongsTo）、Hook、Transaction，MUST 附程式碼範例。

#### Scenario: GORM CRUD 範例
- **WHEN** 讀者閱讀 GORM 章節
- **THEN** 文章提供從 Model 定義到 CRUD 操作的完整範例

#### Scenario: GORM 關聯查詢
- **WHEN** 讀者閱讀關聯章節
- **THEN** 文章展示 HasMany 關聯的定義和 Preload 查詢

### Requirement: sqlx 介紹
文章 SHALL 介紹 sqlx 的核心功能：StructScan、NamedQuery、Get/Select、In 語法，MUST 用與 GORM 相同的資料模型來對比。

#### Scenario: sqlx CRUD 範例
- **WHEN** 讀者閱讀 sqlx 章節
- **THEN** 文章提供與 GORM 等價操作的 sqlx 實作

### Requirement: GORM vs sqlx 對比
文章 SHALL 包含對比分析，涵蓋抽象程度、效能、學習曲線、適用場景。

#### Scenario: 選擇建議
- **WHEN** 讀者閱讀對比章節
- **THEN** 文章給出不同場景下的選擇建議（快速開發 vs 效能敏感 vs SQL 控制力）

### Requirement: 系列導覽
文章開頭 SHALL 有引言，末尾 SHALL 有「下一步」導覽到 PostgreSQL 篇。

#### Scenario: 系列連貫性
- **WHEN** 讀者開啟文章
- **THEN** 開頭有引言段落，末尾引導到下一篇
