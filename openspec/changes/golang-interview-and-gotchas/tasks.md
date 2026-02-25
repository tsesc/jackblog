## 1. Go 面試必問問題文章（第十一篇）

- [x] 1.1 建立 `content/posts/golang-series-11-interview-questions.md`，設定正確的 front matter（title、date 2026-02-25T11:00:00+08:00、tags 含 'Golang系列'、weight = 1）
- [x] 1.2 撰寫文章開頭引言與 Goroutine/Channel 面試題（GMP 模型、channel 阻塞行為、select 用法），附程式碼範例
- [x] 1.3 撰寫記憶體管理面試題（stack vs heap、escape analysis、GC 機制），附程式碼範例與 gcflags 示範
- [x] 1.4 撰寫 Interface 面試題（iface/eface 底層結構、type assertion、type switch），附程式碼範例
- [x] 1.5 撰寫 Slice 底層原理面試題（slice header、append 擴容機制、與底層 array 關係），附程式碼範例
- [x] 1.6 撰寫 Map 面試題（hash table 實作、非 thread-safe、sync.Map），附程式碼範例
- [x] 1.7 撰寫 Defer/Panic/Recover 面試題（LIFO 順序、具名回傳值、recover 模式），附程式碼範例
- [x] 1.8 撰寫文章末尾「下一步」導覽，連結到第十二篇

## 2. Go 常見陷阱與反模式文章（第十二篇）

- [x] 2.1 建立 `content/posts/golang-series-12-gotchas.md`，設定正確的 front matter（title、date 2026-02-25T12:00:00+08:00、tags 含 'Golang系列'、weight = 1）
- [x] 2.2 撰寫 Slice 陷阱（append 修改原始資料、截取不複製、記憶體洩漏），附錯誤與正確對照範例
- [x] 2.3 撰寫 Closure 變數捕獲陷阱（for loop 經典問題、Go 1.22 loopvar 修正），附程式碼範例
- [x] 2.4 撰寫 Nil interface 陷阱（typed nil pointer 不等於 nil、interface 內部結構），附程式碼範例
- [x] 2.5 撰寫 Map 併發陷阱（併發讀寫 panic、range 期間修改），附程式碼範例與正確做法
- [x] 2.6 撰寫 String/Byte 陷阱（string 不可變、range 按 rune 迭代、轉換效能），附程式碼範例
- [x] 2.7 撰寫 Goroutine 洩漏陷阱（channel 未關閉、context 未傳遞、缺少 timeout），附程式碼範例
- [x] 2.8 撰寫 Error 處理陷阱（忽略回傳值、== vs errors.Is、自定義 error），附程式碼範例
- [x] 2.9 撰寫 Defer 陷阱（迴圈中 defer、參數求值時機、錯誤處理），附程式碼範例
- [x] 2.10 撰寫文章末尾系列回顧總結
