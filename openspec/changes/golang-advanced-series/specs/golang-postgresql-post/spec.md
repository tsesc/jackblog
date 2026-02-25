## ADDED Requirements

### Requirement: PostgreSQL 文章結構
文章 SHALL 使用 Hugo front matter，title 為「Go 語言系列（十五）：PostgreSQL 進階」，date 為 2026-02-25T15:00:00+08:00，tags 包含 'Golang系列'，weight = 1。

#### Scenario: Front matter 格式正確
- **WHEN** 文章被 Hugo 讀取
- **THEN** front matter 包含正確的 title、date、tags（含 'Golang系列'）、categories='技術筆記'、author='Jack'、toc=true、weight=1

### Requirement: 從 SQLite 到 PostgreSQL
文章 SHALL 說明為什麼要從 SQLite 升級到 PostgreSQL，以及 pgx driver 的使用方式。MUST 提供 Docker Compose 快速啟動 PostgreSQL 的配置。

#### Scenario: Docker Compose 啟動
- **WHEN** 讀者閱讀環境設定章節
- **THEN** 文章提供可直接使用的 docker-compose.yml 啟動 PostgreSQL

### Requirement: Database Migration
文章 SHALL 介紹 golang-migrate 工具，涵蓋 migration 檔案命名規則、up/down migration、CLI 操作與程式碼整合。

#### Scenario: Migration 操作
- **WHEN** 讀者閱讀 migration 章節
- **THEN** 文章展示建立、執行和回滾 migration 的完整流程

### Requirement: 連線池與效能
文章 SHALL 介紹 database/sql 的連線池設定（MaxOpenConns、MaxIdleConns、ConnMaxLifetime）和 pgx 的 pool 配置。

#### Scenario: 連線池設定
- **WHEN** 讀者閱讀連線池章節
- **THEN** 文章提供生產環境建議的連線池參數和說明

### Requirement: PostgreSQL 進階功能
文章 SHALL 介紹 PostgreSQL 特有功能：Index 策略（B-tree、GIN）、Transaction isolation level、JSONB 欄位操作。

#### Scenario: JSONB 查詢
- **WHEN** 讀者閱讀進階功能章節
- **THEN** 文章展示在 Go 中操作 PostgreSQL JSONB 欄位的程式碼範例

### Requirement: 系列導覽
文章開頭 SHALL 有引言，末尾 SHALL 有「下一步」導覽到 gRPC 篇。

#### Scenario: 系列連貫性
- **WHEN** 讀者開啟文章
- **THEN** 開頭有引言段落，末尾引導到下一篇
