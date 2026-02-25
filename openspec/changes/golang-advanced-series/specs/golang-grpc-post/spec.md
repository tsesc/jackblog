## ADDED Requirements

### Requirement: gRPC 文章結構
文章 SHALL 使用 Hugo front matter，title 為「Go 語言系列（十六）：gRPC 入門」，date 為 2026-02-25T16:00:00+08:00，tags 包含 'Golang系列'，weight = 1。

#### Scenario: Front matter 格式正確
- **WHEN** 文章被 Hugo 讀取
- **THEN** front matter 包含正確的 title、date、tags（含 'Golang系列'）、categories='技術筆記'、author='Jack'、toc=true、weight=1

### Requirement: gRPC 與 REST 對比
文章 SHALL 說明 gRPC 相比 REST 的優勢（效能、強型別、streaming），以及適用場景。

#### Scenario: 對比分析
- **WHEN** 讀者閱讀開頭章節
- **THEN** 文章用表格對比 gRPC 和 REST 的差異

### Requirement: Protocol Buffers
文章 SHALL 介紹 protobuf 的定義語法、安裝 protoc 和 Go plugin、程式碼生成流程。

#### Scenario: Proto 定義與生成
- **WHEN** 讀者閱讀 protobuf 章節
- **THEN** 文章提供完整的 .proto 定義和 protoc 生成指令

### Requirement: Unary RPC 實作
文章 SHALL 提供完整的 Unary RPC 範例，包含 server 和 client 端的實作。

#### Scenario: Unary RPC 範例
- **WHEN** 讀者閱讀 Unary RPC 章節
- **THEN** 文章提供可運行的 server + client 程式碼

### Requirement: Streaming RPC
文章 SHALL 介紹 Server Streaming、Client Streaming、Bidirectional Streaming，MUST 至少提供一種 streaming 的完整範例。

#### Scenario: Server Streaming 範例
- **WHEN** 讀者閱讀 streaming 章節
- **THEN** 文章展示 server streaming 的 proto 定義和 Go 實作

### Requirement: gRPC 攔截器
文章 SHALL 介紹 Unary 和 Stream Interceptor，展示如何實現日誌、認證等橫切關注點。

#### Scenario: 攔截器範例
- **WHEN** 讀者閱讀攔截器章節
- **THEN** 文章提供 logging interceptor 的實作範例

### Requirement: 系列導覽
文章開頭 SHALL 有引言，末尾 SHALL 有「下一步」導覽到微服務篇。

#### Scenario: 系列連貫性
- **WHEN** 讀者開啟文章
- **THEN** 開頭有引言段落，末尾引導到下一篇
