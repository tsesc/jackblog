## Context

Golang 系列已有 12 篇文章（基礎 5 篇 + 實戰 5 篇 + 面試/陷阱 2 篇），使用 Hugo + PaperMod 主題，繁體中文撰寫。所有文章的 front matter 使用 weight = 1，靠 date 的小時差來控制系列排序。現有文章日期為 2026-02-24（01:00-10:00）和 2026-02-25（11:00-12:00）。

## Goals / Non-Goals

**Goals:**
- 延伸系列到 18 篇，涵蓋 Go 進階生態工具與架構模式
- 每篇文章獨立可讀，但整體有遞進關係
- 提供實際可運行的程式碼範例
- 保持與前 12 篇一致的風格和格式

**Non-Goals:**
- 不寫完整的專案教學（每篇聚焦單一主題的核心概念）
- 不深入雲端平台特定操作（AWS/GCP/Azure 細節）
- 不涵蓋前端或 UI 相關內容

## Decisions

### 1. 文章日期與排序
- 日期統一使用 2026-02-25，小時從 13:00 到 18:00（接續前兩篇的 11:00、12:00）
- weight = 1（與所有其他文章一致）
- 這確保在 tag 頁面上，進階系列排在面試/陷阱篇之後

### 2. Web 框架選擇：同時介紹 Gin 和 Echo
- Gin 是目前最流行的 Go Web 框架，Echo 是效能和設計上的強勁對手
- 對比介紹讓讀者能根據需求選擇，而非只學一個
- 以同一個 API 範例分別用兩個框架實作，直觀對比

### 3. ORM 篇：GORM + sqlx 並列
- GORM 是功能最完整的 Go ORM，適合快速開發
- sqlx 是 database/sql 的輕量擴展，適合偏好寫 SQL 的開發者
- 兩者代表不同哲學（full ORM vs SQL-first），並列讓讀者了解取捨

### 4. PostgreSQL 篇：以 golang-migrate 做 Migration
- golang-migrate 是 Go 生態中最廣泛使用的 migration 工具
- 搭配 pgx 作為 PostgreSQL driver（效能優於 lib/pq）

### 5. gRPC 篇：從 Unary 到 Streaming
- 先建立 Unary RPC 的基礎認知
- 再介紹 Server/Client/Bidirectional Streaming
- 包含 protobuf 定義和程式碼生成流程

### 6. 微服務篇：概念與模式為主
- 聚焦架構模式（API Gateway、Service Discovery、Circuit Breaker）
- 用簡化的範例展示服務拆分和通訊
- 不依賴特定框架（go-kit、go-micro 僅作提及）

### 7. Kubernetes 篇：從 Docker 到 K8s
- 銜接第十篇的 Docker 內容
- 涵蓋 Deployment、Service、ConfigMap/Secret、Health Check
- 包含 Helm chart 基礎

## Risks / Trade-offs

- **範例程式碼長度**：進階主題的範例可能較長 → 每個主題精選 2-3 個核心範例，避免面面俱到
- **版本依賴**：框架和工具版本更新快 → 標注使用的版本，讀者可自行調整
- **PostgreSQL 實際操作**：讀者需要安裝 PostgreSQL → 提供 Docker Compose 快速啟動方式
- **K8s 環境門檻**：讀者需要 K8s 叢集 → 推薦使用 minikube 或 kind 作為本地環境
