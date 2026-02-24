+++
title = 'Go 語言系列（二）：環境設定與 Hello World'
date = 2026-02-26T10:00:00+08:00
draft = false
tags = ['Golang', 'Go', '環境設定', '開發工具', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '手把手帶你安裝 Go 開發環境、設定 VS Code 編輯器、理解 Go Modules，並寫出第一支 Go 程式'
toc = true
weight = 9
+++

> 這是 **Go 語言從零到 Web 應用**系列的第二篇。上一篇我們了解了為什麼要學 Go，這篇要動手把環境準備好，並寫出你的第一支 Go 程式。

## 安裝 Go

### macOS

使用 Homebrew 是最方便的方式：

```bash
brew install go
```

或者從 [官方下載頁面](https://go.dev/dl/) 下載 `.pkg` 安裝檔，雙擊安裝即可。

### Windows

從 [官方下載頁面](https://go.dev/dl/) 下載 `.msi` 安裝檔，雙擊後按照精靈指示安裝。安裝程式會自動設定環境變數。

### Linux

```bash
# 下載最新版本（以 1.22 為例）
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz

# 解壓到 /usr/local
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# 加入 PATH（加到 ~/.bashrc 或 ~/.zshrc）
export PATH=$PATH:/usr/local/go/bin
```

### 驗證安裝

安裝完成後，開啟終端機執行：

```bash
go version
```

你應該會看到類似以下的輸出：

```
go version go1.22.0 darwin/arm64
```

## 認識 Go 的目錄結構

Go 有幾個重要的環境變數：

```bash
# 查看 Go 的環境設定
go env
```

最重要的兩個：

- **GOROOT**：Go 的安裝目錄（通常不需要手動設定）
- **GOPATH**：Go 的工作空間，預設是 `$HOME/go`

在 GOPATH 下有三個目錄：

```
$HOME/go/
├── bin/    # 編譯後的執行檔
├── pkg/    # 編譯後的套件快取
└── src/    # 原始碼（現代 Go 較少使用）
```

確保 `$GOPATH/bin` 在你的 PATH 中：

```bash
# 加到 ~/.bashrc、~/.zshrc 或 ~/.config/fish/config.fish
export PATH=$PATH:$(go env GOPATH)/bin
```

## 設定 VS Code

### 安裝 Go 擴充套件

1. 開啟 VS Code
2. 按 `Cmd+Shift+X`（macOS）或 `Ctrl+Shift+X`（Windows/Linux）開啟擴充套件面板
3. 搜尋 **Go**（由 Go Team at Google 發布）
4. 點擊安裝

### 安裝 Go 工具

安裝完擴充套件後，按 `Cmd+Shift+P` 開啟命令面板，輸入 `Go: Install/Update Tools`，全選後安裝。這會安裝以下工具：

- **gopls**：Go 語言伺服器，提供自動補全、跳轉定義等功能
- **dlv**：偵錯器
- **staticcheck**：靜態分析工具
- **goimports**：自動管理 import 語句

### 推薦的 VS Code 設定

在 `.vscode/settings.json` 中加入：

```json
{
    "go.formatTool": "goimports",
    "go.lintTool": "staticcheck",
    "editor.formatOnSave": true,
    "[go]": {
        "editor.defaultFormatter": "golang.go",
        "editor.codeActionsOnSave": {
            "source.organizeImports": "explicit"
        }
    }
}
```

這樣每次存檔時，VS Code 會自動格式化程式碼並整理 import。

## Go Modules 基礎

Go Modules 是 Go 的官方依賴管理系統（Go 1.11+ 引入，Go 1.16+ 預設啟用）。每個 Go 專案都是一個 module。

### 建立第一個專案

```bash
# 建立專案目錄
mkdir hello-go
cd hello-go

# 初始化 Go module
go mod init hello-go
```

`go mod init` 會建立一個 `go.mod` 檔案：

```
module hello-go

go 1.22
```

這個檔案記錄了：
- **module 名稱**：你的專案名稱
- **Go 版本**：最低相容的 Go 版本
- 之後還會記錄外部依賴

## Hello World

### 撰寫程式碼

在專案目錄中建立 `main.go`：

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

讓我們逐行解讀：

- **`package main`**：宣告這個檔案屬於 `main` 套件。`main` 套件是特殊的——它表示這是一個可執行程式（而非函式庫）
- **`import "fmt"`**：引入標準庫的 `fmt` 套件，提供格式化輸入/輸出功能
- **`func main()`**：程式的進入點。`main` 套件中的 `main` 函式是 Go 程式執行的起點
- **`fmt.Println("Hello, World!")`**：印出一行文字並換行

### 執行程式

有兩種方式：

**方式一：直接執行**

```bash
go run main.go
```

輸出：
```
Hello, World!
```

`go run` 會在背後編譯並立即執行，適合開發時快速測試。

**方式二：編譯後執行**

```bash
# 編譯
go build -o hello

# 執行
./hello
```

`go build` 會產生一個獨立的二進位執行檔。這個檔案不需要 Go runtime 就能執行，可以直接複製到其他同作業系統的機器上運行。

## 常用的 Go 指令

### go run

```bash
# 執行單一檔案
go run main.go

# 執行目前目錄下的所有 Go 檔案
go run .
```

### go build

```bash
# 編譯目前目錄
go build

# 指定輸出檔名
go build -o myapp

# 跨平台編譯
GOOS=linux GOARCH=amd64 go build -o myapp-linux
GOOS=windows GOARCH=amd64 go build -o myapp.exe
GOOS=darwin GOARCH=arm64 go build -o myapp-mac
```

跨平台編譯是 Go 的一大亮點——一行指令就能產生不同平台的執行檔。

### go mod

```bash
# 初始化新 module
go mod init <module-name>

# 下載依賴
go mod download

# 整理依賴（移除未使用的，加入缺少的）
go mod tidy

# 查看依賴
go list -m all
```

### go fmt

```bash
# 格式化單一檔案
go fmt main.go

# 格式化整個專案
go fmt ./...
```

`./...` 是 Go 的萬用字元，表示「目前目錄及所有子目錄」。

### go vet

```bash
# 靜態分析，找出常見錯誤
go vet ./...
```

`go vet` 能幫你找到編譯器抓不到的問題，例如格式化字串與參數不匹配等。

### go test

```bash
# 執行所有測試
go test ./...

# 執行測試並顯示詳細輸出
go test -v ./...
```

我們在後面的文章中會詳細介紹測試。

## 練習：小試身手

試著修改 `main.go`，讓它做更多事情：

```go
package main

import "fmt"

func main() {
    // 宣告變數
    name := "Go"
    version := 1.22

    // 格式化輸出
    fmt.Printf("Hello, %s!\n", name)
    fmt.Printf("你正在使用 Go %.2f\n", version)

    // 簡單的計算
    a, b := 10, 20
    fmt.Printf("%d + %d = %d\n", a, b, a+b)
}
```

執行看看結果：

```bash
go run main.go
```

輸出：
```
Hello, Go!
你正在使用 Go 1.22
10 + 20 = 30
```

注意幾個重點：

- `:=` 是 Go 的短變數宣告，會自動推斷型別
- `Printf` 使用格式化佔位符（`%s` 字串、`%f` 浮點數、`%d` 整數）
- Go 支援同時宣告多個變數（`a, b := 10, 20`）

## 專案結構建議

雖然 Hello World 只需要一個檔案，但了解基本的專案結構很重要：

```
hello-go/
├── go.mod          # Module 定義
├── go.sum          # 依賴的 checksum（有外部依賴時自動產生）
├── main.go         # 程式進入點
└── README.md       # 專案說明
```

隨著專案成長，你會開始拆分成多個檔案和目錄。我們會在第七篇文章中詳細討論 Web 專案的結構。

## 常見問題

### Q: `go: command not found`

Go 沒有正確加入 PATH。確認安裝目錄在 PATH 中：

```bash
# macOS/Linux
echo $PATH | tr ':' '\n' | grep go
```

### Q: VS Code 沒有自動補全

確認 gopls 已安裝：

```bash
go install golang.org/x/tools/gopls@latest
```

然後重新啟動 VS Code。

### Q: `go mod init` 的模組名稱要怎麼取？

- 個人專案：隨意取，例如 `hello-go`
- 要發布的套件：使用 repository URL，例如 `github.com/username/project`

## 下一步

環境準備好了，Hello World 也跑起來了！在下一篇文章中，我們將深入 Go 的基礎語法——變數、型別、條件判斷、迴圈和函式。這些是寫任何 Go 程式的基本功。
