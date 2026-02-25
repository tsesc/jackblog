## ADDED Requirements

### Requirement: 微服務文章結構
文章 SHALL 使用 Hugo front matter，title 為「Go 語言系列（十七）：微服務架構」，date 為 2026-02-25T17:00:00+08:00，tags 包含 'Golang系列'，weight = 1。

#### Scenario: Front matter 格式正確
- **WHEN** 文章被 Hugo 讀取
- **THEN** front matter 包含正確的 title、date、tags（含 'Golang系列'）、categories='技術筆記'、author='Jack'、toc=true、weight=1

### Requirement: 單體 vs 微服務
文章 SHALL 分析單體架構和微服務架構的優缺點，說明何時該拆分微服務。

#### Scenario: 架構對比
- **WHEN** 讀者閱讀開頭章節
- **THEN** 文章提供單體和微服務的對比表，以及拆分時機的判斷準則

### Requirement: 服務拆分策略
文章 SHALL 介紹 DDD（Domain-Driven Design）的 Bounded Context 概念，展示如何根據業務領域拆分服務。

#### Scenario: 拆分範例
- **WHEN** 讀者閱讀拆分章節
- **THEN** 文章用一個電商系統範例展示如何拆分成 User、Order、Product 服務

### Requirement: 服務間通訊
文章 SHALL 介紹同步通訊（HTTP/gRPC）和非同步通訊（Message Queue）的模式與取捨。

#### Scenario: 通訊模式範例
- **WHEN** 讀者閱讀通訊章節
- **THEN** 文章展示 HTTP 和 gRPC 呼叫其他服務的 Go 程式碼範例

### Requirement: 微服務模式
文章 SHALL 介紹核心微服務模式：API Gateway、Service Discovery、Circuit Breaker、Distributed Tracing。

#### Scenario: Circuit Breaker 範例
- **WHEN** 讀者閱讀模式章節
- **THEN** 文章展示使用 Go 實現或使用現有 library 的 circuit breaker 範例

### Requirement: 系列導覽
文章開頭 SHALL 有引言，末尾 SHALL 有「下一步」導覽到 Kubernetes 篇。

#### Scenario: 系列連貫性
- **WHEN** 讀者開啟文章
- **THEN** 開頭有引言段落，末尾引導到下一篇
