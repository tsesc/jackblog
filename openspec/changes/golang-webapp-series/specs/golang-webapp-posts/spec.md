## ADDED Requirements

### Requirement: HTTP 基礎與第一個 Web Server 文章
系統 SHALL 包含一篇 HTTP 基礎文章（`golang-series-06-http-basics.md`），涵蓋 `net/http` 標準庫、Handler 與 HandlerFunc、路由基礎、Request 和 Response 物件、提供靜態檔案，以及 JSON 回應。

#### Scenario: 讀者建立第一個 HTTP Server
- **WHEN** 讀者按照文章步驟操作
- **THEN** 讀者能啟動一個處理多個路由的 HTTP server，回傳 JSON 回應

### Requirement: 專案結構與路由文章
系統 SHALL 包含一篇專案結構文章（`golang-series-07-project-structure.md`），涵蓋 Go Web 專案的標準目錄結構、使用 Go 1.22+ 增強路由或 chi router、中間件（Middleware）概念與實作（logging、recovery）、請求參數解析（path params、query params、body），以及環境變數與設定管理。

#### Scenario: 讀者建立結構化的 Web 專案
- **WHEN** 讀者按照文章步驟操作
- **THEN** 讀者能建立具有清晰目錄結構、路由分組、中間件的 Web 專案

### Requirement: 資料庫整合文章
系統 SHALL 包含一篇資料庫整合文章（`golang-series-08-database.md`），涵蓋 `database/sql` 標準介面、SQLite 連接與設定、資料表建立與 Migration 概念、CRUD 操作實作、SQL Injection 防護（prepared statements），以及 Repository Pattern 的應用。

#### Scenario: 讀者完成資料庫 CRUD 功能
- **WHEN** 讀者按照文章步驟操作
- **THEN** 讀者能在 Web 應用中實現對資料庫的新增、讀取、更新、刪除操作

### Requirement: 使用者認證文章
系統 SHALL 包含一篇認證文章（`golang-series-09-authentication.md`），涵蓋密碼雜湊（bcrypt）、JWT 基礎與實作、認證中間件、受保護的 API 端點，以及使用者註冊與登入流程。

#### Scenario: 讀者實現完整的認證流程
- **WHEN** 讀者按照文章步驟操作
- **THEN** 讀者能在 Web 應用中實現使用者註冊、登入、JWT token 發放與驗證

### Requirement: 測試與部署文章
系統 SHALL 包含一篇測試與部署文章（`golang-series-10-testing-and-deployment.md`），涵蓋 Go 內建測試框架（`testing` package）、單元測試與表格驅動測試、HTTP Handler 測試（httptest）、Docker 化部署、完整專案回顧與總結。

#### Scenario: 讀者完成測試與容器化部署
- **WHEN** 讀者按照文章步驟操作
- **THEN** 讀者能為 Web 應用撰寫測試，並使用 Docker 打包部署應用程式

#### Scenario: 讀者回顧整個系列
- **WHEN** 讀者閱讀完最後一篇的總結
- **THEN** 讀者能完整理解從 Go 基礎到 Web 應用開發的完整流程，具備獨立開發 Go Web 應用的能力
