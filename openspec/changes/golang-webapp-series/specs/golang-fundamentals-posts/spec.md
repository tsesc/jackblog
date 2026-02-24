## ADDED Requirements

### Requirement: Go 語言介紹文章
系統 SHALL 包含一篇「為什麼要學 Go」的介紹文章（`golang-series-01-why-golang.md`），涵蓋 Go 的歷史背景、設計哲學、適用場景、與其他語言的比較，以及 Go 在業界的應用案例。

#### Scenario: 讀者了解 Go 的定位
- **WHEN** 讀者閱讀完第一篇文章
- **THEN** 讀者能理解 Go 語言的核心優勢（簡潔、高效能、併發原生支持）及其最適合的應用場景（後端服務、CLI 工具、雲端基礎設施）

### Requirement: 環境設定與第一支程式文章
系統 SHALL 包含一篇環境設定文章（`golang-series-02-setup-and-hello-world.md`），涵蓋 Go 安裝、開發環境設定（VS Code + Go 擴充）、Go modules 基礎、第一支 Hello World 程式，以及 `go run`、`go build`、`go mod` 基本指令。

#### Scenario: 讀者成功執行第一支 Go 程式
- **WHEN** 讀者按照文章步驟操作
- **THEN** 讀者能在本機安裝 Go、建立專案、並成功執行 Hello World 程式

### Requirement: 基礎語法文章
系統 SHALL 包含一篇基礎語法文章（`golang-series-03-basic-syntax.md`），涵蓋變數宣告（var、:=）、基本型別（int、string、bool、float）、條件判斷（if/else、switch）、迴圈（for）、函式定義與多值回傳、指標基礎。

#### Scenario: 讀者掌握基本語法
- **WHEN** 讀者閱讀完語法文章
- **THEN** 讀者能撰寫包含變數、條件、迴圈、函式的 Go 程式

### Requirement: 資料結構文章
系統 SHALL 包含一篇資料結構文章（`golang-series-04-data-structures.md`），涵蓋 Array 與 Slice、Map、Struct、Interface 基礎、Method（值接收者與指標接收者），以及組合（Composition）vs 繼承的概念。

#### Scenario: 讀者能使用 Go 的核心資料結構
- **WHEN** 讀者閱讀完資料結構文章
- **THEN** 讀者能定義 struct、實作 interface、使用 slice 和 map 處理資料集合

### Requirement: 錯誤處理與併發文章
系統 SHALL 包含一篇錯誤處理與併發文章（`golang-series-05-errors-and-concurrency.md`），涵蓋 Go 的錯誤處理模式（error interface、自訂 error）、defer/panic/recover、Goroutine 基礎、Channel 通訊、sync.WaitGroup，以及常見併發模式。

#### Scenario: 讀者理解 Go 的錯誤處理與併發模型
- **WHEN** 讀者閱讀完此文章
- **THEN** 讀者能撰寫正確的錯誤處理程式碼，並使用 goroutine 和 channel 實現基本的併發操作
