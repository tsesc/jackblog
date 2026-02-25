## ADDED Requirements

### Requirement: Go 常見陷阱文章結構
文章 SHALL 使用 Hugo front matter 格式，title 為「Go 語言系列（十二）：常見陷阱與反模式」，date 為 2026-02-25T12:00:00+08:00，tags 包含 'Golang系列'，categories 為 '技術筆記'，weight = 1。

#### Scenario: Front matter 格式正確
- **WHEN** 文章檔案被 Hugo 讀取
- **THEN** front matter 包含正確的 title、date、tags（含 'Golang系列'）、categories、author、description、toc = true、weight = 1

### Requirement: Slice 陷阱
文章 SHALL 涵蓋 Slice 相關陷阱，包含：append 可能修改原始 slice 的底層 array、slice 截取不會複製資料、大 slice 截取小片段導致記憶體無法釋放，每個陷阱 MUST 附帶「錯誤範例」和「正確做法」的對照程式碼。

#### Scenario: Append 修改原始 slice
- **WHEN** 讀者閱讀 Slice 陷阱
- **THEN** 文章展示 append 到未滿 capacity 的 slice 會修改原始資料的範例，並提供 copy 或 full slice expression 的正確做法

#### Scenario: 記憶體洩漏
- **WHEN** 讀者閱讀 Slice 記憶體陷阱
- **THEN** 文章展示從大 slice 截取小片段導致大 array 無法被 GC 的範例

### Requirement: Closure 變數捕獲陷阱
文章 SHALL 涵蓋 closure 在 for loop 中捕獲迴圈變數的經典陷阱，MUST 說明 Go 1.22 之前與之後的行為差異。

#### Scenario: For loop closure 陷阱
- **WHEN** 讀者閱讀 Closure 陷阱
- **THEN** 文章展示 goroutine 在 for loop 中捕獲到相同變數的問題，並說明 Go 1.22 的 loopvar 修正

### Requirement: Nil interface 陷阱
文章 SHALL 涵蓋 nil interface 的判斷陷阱：一個持有 nil typed pointer 的 interface 不等於 nil，MUST 解釋 interface 內部的 (type, value) 結構。

#### Scenario: Nil interface 不等於 nil
- **WHEN** 讀者閱讀 nil interface 陷阱
- **THEN** 文章展示將 nil pointer 賦值給 interface 後，`if err != nil` 判斷為 true 的範例

### Requirement: Map 併發陷阱
文章 SHALL 涵蓋 map 在併發場景下的陷阱，包含：併發讀寫 panic、range map 期間修改的行為，MUST 提供 sync.Mutex 或 sync.Map 的正確做法。

#### Scenario: 併發讀寫 panic
- **WHEN** 讀者閱讀 Map 併發陷阱
- **THEN** 文章展示多個 goroutine 同時讀寫 map 導致 `fatal error: concurrent map writes` 的範例

### Requirement: String 與 Byte 陷阱
文章 SHALL 涵蓋 string 和 []byte 的常見陷阱，包含：string 是不可變的、range string 是按 rune 而非 byte 迭代、string 與 []byte 轉換的效能開銷。

#### Scenario: Range string 按 rune 迭代
- **WHEN** 讀者閱讀 String 陷阱
- **THEN** 文章展示 range 中文字串時 index 跳躍的行為，並解釋 rune vs byte 的差異

### Requirement: Goroutine 洩漏陷阱
文章 SHALL 涵蓋 goroutine 洩漏的常見模式，包含：忘記關閉 channel、context 未正確傳遞、select 中缺少 default 或 timeout，MUST 提供預防方法。

#### Scenario: Channel 未關閉導致洩漏
- **WHEN** 讀者閱讀 Goroutine 洩漏陷阱
- **THEN** 文章展示 goroutine 因等待永遠不會收到資料的 channel 而洩漏的範例，並提供 context.WithCancel 的正確做法

### Requirement: Error 處理陷阱
文章 SHALL 涵蓋 error 處理的常見陷阱，包含：忽略 error 回傳值、error 比較使用 == 而非 errors.Is、自定義 error type 的 pointer receiver 問題。

#### Scenario: Error 比較
- **WHEN** 讀者閱讀 Error 處理陷阱
- **THEN** 文章展示使用 == 比較 wrapped error 失敗的範例，並說明 errors.Is 和 errors.As 的正確用法

### Requirement: Defer 陷阱
文章 SHALL 涵蓋 defer 的常見陷阱，包含：defer 在迴圈中造成資源堆積、defer 的參數在呼叫時就已求值、defer 中的錯誤處理。

#### Scenario: Defer 參數求值時機
- **WHEN** 讀者閱讀 Defer 陷阱
- **THEN** 文章展示 defer 的參數在 defer 語句執行時（非函式結束時）就已經求值的範例

#### Scenario: 迴圈中的 defer
- **WHEN** 讀者閱讀迴圈 defer 陷阱
- **THEN** 文章展示在 for loop 中 defer close 檔案導致 file descriptor 耗盡的範例

### Requirement: 文章導覽連結
文章開頭 SHALL 包含系列導覽文字，說明此為系列第十二篇，也是系列的最終篇。

#### Scenario: 系列完結
- **WHEN** 讀者開啟文章
- **THEN** 文章開頭有引言段落說明此為系列最終篇，末尾有整個系列的回顧總結
