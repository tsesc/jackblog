## ADDED Requirements

### Requirement: Go 面試必問問題文章結構
文章 SHALL 使用 Hugo front matter 格式，title 為「Go 語言系列（十一）：面試必問問題」，date 為 2026-02-25T11:00:00+08:00，tags 包含 'Golang系列'，categories 為 '技術筆記'，weight = 1。

#### Scenario: Front matter 格式正確
- **WHEN** 文章檔案被 Hugo 讀取
- **THEN** front matter 包含正確的 title、date、tags（含 'Golang系列'）、categories、author、description、toc = true、weight = 1

### Requirement: Goroutine 與 Channel 面試題
文章 SHALL 包含 Goroutine 與 Channel 相關面試題，涵蓋 goroutine 排程機制（GMP 模型）、channel 的底層實作與阻塞行為、select 語句的使用場景，每題 MUST 附帶程式碼範例與解析。

#### Scenario: GMP 模型解析
- **WHEN** 讀者閱讀 Goroutine 排程部分
- **THEN** 文章解釋 G（Goroutine）、M（Machine/OS Thread）、P（Processor）的角色與互動

#### Scenario: Channel 阻塞行為
- **WHEN** 讀者閱讀 Channel 面試題
- **THEN** 文章透過程式碼範例展示 unbuffered 和 buffered channel 的阻塞差異

### Requirement: 記憶體管理面試題
文章 SHALL 包含 Go 記憶體管理面試題，涵蓋 stack vs heap 分配、escape analysis、garbage collector 機制，每題 MUST 附帶程式碼範例。

#### Scenario: Escape analysis 範例
- **WHEN** 讀者閱讀記憶體管理部分
- **THEN** 文章提供程式碼範例展示變數何時逃逸到 heap，並示範如何用 `go build -gcflags="-m"` 檢查

### Requirement: Interface 面試題
文章 SHALL 包含 Interface 相關面試題，涵蓋 interface 的底層結構（iface/eface）、型別斷言（type assertion）與 type switch、空 interface 的使用與限制。

#### Scenario: Interface 底層結構
- **WHEN** 讀者閱讀 Interface 面試題
- **THEN** 文章解釋 iface（帶方法的 interface）和 eface（空 interface）的記憶體佈局差異

### Requirement: Slice 底層原理面試題
文章 SHALL 包含 Slice 底層原理面試題，涵蓋 slice header 結構（pointer、length、capacity）、append 的擴容機制、slice 與底層 array 的關係。

#### Scenario: Slice 擴容機制
- **WHEN** 讀者閱讀 Slice 面試題
- **THEN** 文章透過程式碼範例展示 append 觸發擴容前後 slice 的 capacity 變化

### Requirement: Map 面試題
文章 SHALL 包含 Map 相關面試題，涵蓋 map 的底層 hash table 實作、map 不是 thread-safe 的原因、sync.Map 的使用場景。

#### Scenario: Map 併發安全
- **WHEN** 讀者閱讀 Map 面試題
- **THEN** 文章展示 map 在併發讀寫時會 panic 的程式碼範例，並提供 sync.Map 或 mutex 的解決方案

### Requirement: Defer、Panic、Recover 面試題
文章 SHALL 包含 defer、panic、recover 的面試題，涵蓋 defer 的執行順序（LIFO）、defer 與具名回傳值的互動、panic/recover 的正確使用模式。

#### Scenario: Defer 執行順序
- **WHEN** 讀者閱讀 defer 面試題
- **THEN** 文章透過程式碼範例展示多個 defer 的 LIFO 執行順序，以及 defer 在函式回傳前修改具名回傳值的行為

### Requirement: 文章導覽連結
文章開頭 SHALL 包含系列導覽文字，說明此為系列第十一篇，承接前十篇內容。

#### Scenario: 系列連貫性
- **WHEN** 讀者開啟文章
- **THEN** 文章開頭有引言段落說明系列定位，末尾有「下一步」引導到第十二篇（常見陷阱）
