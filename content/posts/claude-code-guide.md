+++
title = 'Claude Code 完整使用指南：從入門到進階'
date = 2025-08-18T14:00:00+08:00
draft = false
tags = ['AI工具', 'Claude', '開發工具', '程式設計', 'Anthropic']
categories = ['技術筆記']
author = 'Jack'
description = '深入了解 Anthropic 的 Claude Code，從基礎設置到進階工作流程，掌握 AI 輔助編程的最佳實踐'
toc = true
weight = 1
+++

## 什麼是 Claude Code？

Claude Code 是 Anthropic 推出的命令列工具，專為 Agentic Coding（代理式編程）設計。它將 Claude AI 模型原生整合到工程師的編碼工作流程中，不僅是研究項目，更是 Anthropic 內部工程師和研究人員的日常工具。

### 核心特色

- **低階且無偏見**：提供接近原始模型的存取權限，不強制特定工作流程
- **純代理人模型**：具備強大工具，能自主完成任務直到判斷完成
- **代理人式搜尋**：模仿人類探索程式碼的方式，動態理解程式碼庫

## 快速開始

### 安裝與設置

```bash
# 安裝 Claude Code
npm install -g claude-code

# 初始化專案配置
claude init

# 啟動互動模式
claude
```

### 基礎使用

最簡單的使用方式就是直接對話：

```bash
# 詢問程式碼問題
claude "這個函數是做什麼的？" function.js

# 修復錯誤
claude "修復這個 TypeScript 錯誤"

# 生成程式碼
claude "寫一個排序演算法"

# 查看網頁內容
claude "查看 https://docs.example.com 的 API 文件"

# 計畫模式 (Plan Mode)
claude -p "規劃如何重構這個模組"

# 非互動模式 (Headless Mode)
echo "修復所有測試" | claude --headless
```

## 核心功能詳解

### 1. CLAUDE.md 文件配置

CLAUDE.md 是 Claude 啟動時自動讀取的特殊文件，用於記錄專案關鍵資訊。

#### 放置位置優先順序

1. **專案根目錄**：最常用，包含專案特定資訊
2. **父目錄**：適用於 Monorepo 架構
3. **子目錄**：按需載入特定模組資訊
4. **使用者目錄** (`~/.claude/CLAUDE.md`)：全域設定

#### CLAUDE.md 範例

```markdown
# 專案說明
這是一個 React + TypeScript 的電商網站

## 開發指令
- `npm run dev` - 啟動開發伺服器
- `npm test` - 執行測試
- `npm run build` - 建置生產版本

## 程式碼規範
- 使用 ESLint + Prettier
- 元件使用 PascalCase 命名
- 優先使用函數式元件

## 重要檔案
- `/src/api/` - API 整合層
- `/src/components/` - 共用元件
- `/src/pages/` - 頁面元件

## 注意事項
- 不要修改 `/vendor` 目錄下的檔案
- API 金鑰存放在 `.env.local`
```

### 2. 權限管理

Claude Code 預設採保守策略，需要明確授權才能執行可能修改系統的操作。

#### 設定權限的方法

1. **即時允許**：在提示時選擇 "Always allow"
2. **使用指令**：`/permissions` 新增允許項目
3. **編輯設定檔**：`.claude/settings.json`（推薦團隊共享）
4. **CLI 參數**：`--allowedTools` 進行會話特定設定

#### settings.json 範例

```json
{
  "permissions": {
    "allowedTools": [
      "Bash:npm install:*",
      "Bash:git:*",
      "Write:*.test.ts",
      "Edit:src/**/*"
    ],
    "deniedTools": [
      "Bash:rm -rf:*",
      "Write:*.env"
    ]
  }
}
```

### 3. 工具整合

#### Bash 工具
Claude Code 繼承你的 Bash 環境，可使用所有已安裝的工具。

```bash
# 告訴 Claude 可用的自訂工具
echo "可用工具：docker, kubectl, terraform" >> CLAUDE.md
```

#### MCP (Model Context Protocol)
連接多個 MCP 伺服器以擴展功能：

```json
// .mcp.json
{
  "servers": {
    "database": {
      "command": "mcp-postgres",
      "args": ["--connection", "postgresql://localhost/mydb"]
    },
    "github": {
      "command": "mcp-github",
      "args": ["--token", "$GITHUB_TOKEN"]
    }
  }
}
```

#### GitHub CLI 整合
```bash
# 安裝 GitHub CLI
brew install gh

# Claude 可以執行的操作
claude "建立一個新的 issue 描述這個 bug"
claude "開啟 PR 並寫好描述"
claude "查看最新的 PR 評論"
```

### 4. 斜線命令系統

#### 內建斜線命令

```bash
/help           # 顯示幫助資訊
/clear          # 清理對話上下文
/permissions    # 管理權限設定
/init           # 初始化專案配置
/settings       # 編輯設定檔
/history        # 查看對話歷史
/export         # 匯出對話記錄
/model          # 切換模型
/tokens         # 查看 token 使用量
/debug          # 切換除錯模式
```

#### 自訂斜線命令

在 `.claude/commands/` 目錄下建立 Markdown 檔案：

```markdown
<!-- .claude/commands/review.md -->
請審查以下程式碼的品質：
- 檢查潛在的錯誤
- 評估效能問題
- 提出改進建議

檔案：$ARGUMENTS
```

使用方式：
```bash
/review src/main.ts
```

更多自訂命令範例：

```markdown
<!-- .claude/commands/test.md -->
為 $ARGUMENTS 檔案撰寫完整的單元測試

<!-- .claude/commands/optimize.md -->
優化 $ARGUMENTS 的效能，focus on:
- 時間複雜度
- 空間使用
- 可讀性

<!-- .claude/commands/doc.md -->
為 $ARGUMENTS 生成詳細的 API 文件
```

## 進階模式與參數

### 1. 思考模式層級 (Thinking Modes)

Claude Code 提供不同層級的思考模式，分配不同的運算資源：

```bash
# 基礎思考
claude "think 這個架構有什麼問題"

# 深度思考
claude "think hard 如何優化這個演算法"

# 更深入分析
claude "think harder 重新設計整個系統架構"

# 最高層級思考（耗時較長但更全面）
claude "ultrathink 分析這個複雜的並發問題"
```

**思考層級對比**：
- `think` - 快速分析，適合簡單問題
- `think hard` - 中等深度，適合一般設計決策
- `think harder` - 深入分析，適合複雜問題
- `ultrathink` - 最深層分析，適合系統性重大決策

### 2. 計畫模式 (-p, Plan Mode)

計畫模式讓 Claude 專注於規劃而非立即執行：

```bash
# 進入計畫模式
claude -p

# 在計畫模式下規劃
> 幫我規劃如何將這個單體應用拆分為微服務

# 計畫模式的優勢
# - 不會立即修改檔案
# - 專注於架構和策略
# - 產生詳細的執行步驟
# - 適合大型重構前的準備
```

### 3. 網頁內容讀取

Claude Code 可以直接讀取和分析網頁內容：

```bash
# 讀取文件
claude "閱讀 https://reactjs.org/docs/hooks-intro.html 並解釋 Hooks"

# 分析 API 文件
claude "查看 https://api.stripe.com/docs 並實作支付整合"

# 參考 Stack Overflow
claude "參考這個解答 https://stackoverflow.com/questions/... 來解決問題"

# 設定網域白名單（避免重複授權）
/permissions add WebFetch:*.reactjs.org
```

### 4. 非互動模式 (Headless Mode)

適用於自動化和 CI/CD 整合：

```bash
# 基本使用
echo "格式化所有程式碼" | claude --headless

# JSON 輸出模式（適合程式化處理）
claude --headless --output-format stream-json "分析程式碼品質"

# 搭配錯誤碼
claude --headless --exit-on-error "執行測試"

# 批次處理範例
for file in *.js; do
  echo "優化 $file" | claude --headless
done

# CI/CD 整合範例
# .github/workflows/code-review.yml
- name: Claude Code Review
  run: |
    echo "審查 PR 變更" | claude --headless --output-format stream-json
```

### 5. 子代理 (Subagents)

Claude 可以生成子代理來處理特定任務：

```bash
# 讓 Claude 建立子代理驗證解決方案
claude "使用子代理驗證這個加密實作是否安全"

# 平行處理複雜任務
claude "創建多個子代理分別處理前端、後端和資料庫遷移"
```

## 常見工作流程

### 1. 探索、規劃、編碼、提交

```bash
# 步驟 1：探索
claude "閱讀 authentication 相關的程式碼，先不要寫程式"

# 步驟 2：深度規劃
claude "think harder 如何實作 OAuth2.0 登入"

# 步驟 3：實作
claude "根據計畫實作 OAuth2.0"

# 步驟 4：提交
claude "提交程式碼並建立 PR"
```

### 2. 測試驅動開發 (TDD)

```bash
# 寫測試
claude "為購物車功能寫測試案例（TDD 模式）"

# 執行測試確認失敗
claude "執行測試"

# 實作功能
claude "寫程式碼通過測試，不要修改測試"

# 迭代到通過
claude "繼續修正直到所有測試通過"
```

### 3. 視覺化開發

```bash
# 提供設計稿（拖放或貼上圖片）
claude "根據這個設計稿 design.png 實作頁面"

# 直接貼上截圖
# 1. 截圖 (Cmd+Shift+4 on Mac)
# 2. 在 Claude 對話中貼上 (Cmd+V)
claude "修正這個 UI 問題"

# 比對實作與設計
claude "比較 mockup.png 和目前實作的差異"

# 迭代優化
claude "調整樣式使其更接近設計稿"

# 分析圖表或流程圖
claude "根據這個架構圖 architecture.png 實作系統"
```

### 4. 程式碼庫問答

```bash
# 了解架構
claude "這個專案的認證系統是如何運作的？"

# 學習慣例
claude "如何新增一個新的 API 端點？"

# 找尋功能
claude "日誌記錄是在哪裡實作的？"
```

### 5. Git 操作輔助

```bash
# 智慧 commit
claude "分析變更並撰寫 commit message"

# 複雜操作
claude "幫我 rebase 到 main 並解決衝突"

# 歷史搜尋
claude "找出最近修改 auth 模組的 commits"
```

## 進階技巧

### 1. 優化指令

- **具體明確**：清晰的指令能提高成功率
- **提供圖像**：UI 設計、流程圖、錯誤截圖
- **引用檔案**：使用 Tab 自動補全路徑
- **提供 URL**：直接貼上文件連結

### 2. 上下文管理

```bash
# 長對話後清理上下文
/clear

# 中斷操作
# 按 Escape 鍵

# 回溯歷史
# 雙擊 Escape 鍵

# 恢復中斷的對話
claude --resume

# 從特定對話恢復
claude --resume conversation-id
```

### 3. 管道輸入與輸出

```bash
# 管道輸入資料
cat error.log | claude "分析這些錯誤"

# 處理命令輸出
npm test | claude "解釋測試失敗原因"

# 鏈式處理
claude "生成測試資料" | python process.py | claude "分析結果"

# 重定向輸出
claude "生成配置檔" > config.json
```

### 4. Hooks 系統

Claude Code 支援在特定事件觸發自訂腳本：

```json
// .claude/settings.json
{
  "hooks": {
    "pre-commit": "npm test",
    "post-edit": "npm run lint",
    "pre-tool": "echo 'Tool: $TOOL_NAME'",
    "user-prompt-submit": "git status"
  }
}
```

常見 hooks：
- `pre-commit` - 提交前執行
- `post-edit` - 編輯檔案後執行
- `pre-tool` - 工具呼叫前執行
- `user-prompt-submit` - 使用者提交提示後執行

### 5. 環境變數配置

```bash
# 設定環境變數
export CLAUDE_API_KEY="your-key"
export CLAUDE_MODEL="claude-3-opus"
export CLAUDE_MAX_TOKENS=4000

# 專案特定環境
# .env.claude
CLAUDE_DEFAULT_PERMISSIONS="Bash:*,Write:src/**"
CLAUDE_AUTO_CLEAR=true
CLAUDE_THINKING_MODE=hard
```

### 6. 複雜任務管理

使用 Markdown 作為任務清單：

```markdown
<!-- tasks.md -->
## 重構任務清單
- [ ] 提取共用邏輯到 utils
- [ ] 更新測試案例
- [ ] 優化效能瓶頸
- [ ] 更新文件
```

```bash
claude "根據 tasks.md 執行重構任務"
```

### 7. 快捷鍵總覽

互動模式下的鍵盤快捷鍵：

- **Escape** - 中斷當前操作
- **Escape×2** - 回溯歷史，編輯先前提示
- **Tab** - 自動補全檔案路徑
- **Ctrl+C** - 強制終止
- **Ctrl+D** - 退出互動模式
- **↑/↓** - 瀏覽歷史命令
- **Ctrl+R** - 搜尋歷史命令

### 8. 輸出格式控制

```bash
# 標準輸出
claude "列出所有函數"

# JSON 格式（適合程式處理）
claude --output-format json "分析程式碼結構"

# 串流 JSON（即時輸出）
claude --output-format stream-json "執行長時間任務"

# Markdown 格式（預設）
claude --output-format markdown "生成文件"

# 純文字
claude --output-format plain "生成配置檔"
```

### 9. 安全 YOLO 模式

適用於低風險重複任務：

```bash
# 跳過所有權限檢查（謹慎使用）
claude --dangerously-skip-permissions "修復所有 lint 錯誤"
```

## 進階應用

### 1. 模型選擇與配置

```bash
# 使用特定模型
claude --model claude-3-opus "複雜的系統設計"
claude --model claude-3-sonnet "一般編程任務"
claude --model claude-3-haiku "快速簡單查詢"

# 設定 token 限制
claude --max-tokens 8000 "生成詳細文件"

# 監控 token 使用
claude --show-token-usage "分析大型程式碼庫"
```

### 2. 除錯模式

```bash
# 啟用除錯輸出
claude --debug "診斷問題"

# 詳細日誌
claude --verbose "追蹤執行過程"

# 顯示工具呼叫
claude --show-tool-calls "觀察 Claude 的操作"

# 乾跑模式（只顯示將執行的操作）
claude --dry-run "模擬重構過程"
```

### 3. 無頭模式 (Headless Mode)

適用於 CI/CD 整合：

```bash
# 自動化程式碼審查
echo "審查程式碼" | claude --headless

# Pre-commit hook
claude --headless "檢查程式碼品質" --exit-on-error
```

### 4. 批次處理

```bash
# 大規模遷移
claude "為所有 .js 檔案遷移到 TypeScript"

# 平行處理
find . -name "*.tsx" | xargs -P 4 -I {} claude "優化 {}"
```

### 5. 多實例協作

```bash
# Terminal 1: 開發
claude "實作新功能"

# Terminal 2: 測試
claude "為新功能寫測試"

# Terminal 3: 文件
claude "更新 API 文件"
```

### 6. Git Worktree 工作流

```bash
# 建立多個工作樹
git worktree add ../feature-a feature-a
git worktree add ../feature-b feature-b

# 在不同分支同時工作
cd ../feature-a && claude "實作功能 A"
cd ../feature-b && claude "實作功能 B"
```

### 7. IDE 整合

Claude Code 支援與多種 IDE 整合：

```bash
# VS Code 整合
claude --ide vscode "在 VS Code 中開啟修改的檔案"

# 監控檔案變更
claude --watch "監控 src/ 目錄並自動修復錯誤"

# 整合到編輯器工作流
# .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [{
    "label": "Claude Review",
    "type": "shell",
    "command": "claude --headless 'review ${file}'"
  }]
}
```

### 8. 會話管理

```bash
# 列出所有會話
claude --list-sessions

# 儲存當前會話
claude --save-session my-feature

# 載入會話
claude --load-session my-feature

# 刪除會話
claude --delete-session old-session

# 匯出會話歷史
claude --export-session > session-log.md
```

### 9. 檔案監控模式

```bash
# 監控並自動處理變更
claude --watch src/ "自動格式化和修復 lint 錯誤"

# 監控測試檔案
claude --watch "*.test.js" "更新相關的實作程式碼"

# 監控設定檔
claude --watch "*.config.js" "驗證設定並提供優化建議"
```

## 最佳實踐建議

### DO ✅

1. **先規劃再編碼**：讓 Claude 制定計畫
2. **頻繁糾正方向**：主動引導而非被動接受
3. **定期清理上下文**：保持對話聚焦
4. **善用 CLAUDE.md**：記錄專案特性
5. **設定合理權限**：平衡安全與效率

### DON'T ❌

1. **避免模糊指令**：不要說「改進這段程式碼」
2. **不要忽視錯誤**：及時糾正錯誤方向
3. **避免超長對話**：適時使用 `/clear`
4. **不要跳過測試**：確保程式碼品質
5. **避免敏感資訊**：不要讓 Claude 處理密碼、金鑰

## 實際案例

### 案例 1：快速修復 Bug

```bash
# 定位問題
claude "用戶反映登入後立即登出，幫我找出問題"

# 分析日誌
claude "查看 auth.log 最近的錯誤"

# 修復問題
claude "修復 session timeout 的問題"

# 驗證修復
claude "寫測試確保問題已解決"
```

### 案例 2：重構老舊程式碼

```bash
# 分析現狀
claude "分析 legacy/ 目錄的程式碼品質"

# 制定計畫
claude "think 如何逐步重構這些程式碼"

# 逐步執行
claude "第一步：提取重複邏輯"
claude "第二步：加入型別定義"
claude "第三步：更新測試覆蓋"
```

### 案例 3：建立新專案

```bash
# 初始化專案
claude "建立一個 Next.js + TypeScript 專案"

# 設定架構
claude "設定 ESLint、Prettier 和 Husky"

# 建立基礎元件
claude "建立基礎的 Layout 和導航元件"

# 設定 CI/CD
claude "建立 GitHub Actions 工作流程"
```

## 效能優化建議

1. **批次操作**：一次處理多個相關檔案
2. **快取利用**：重複任務使用相同 session
3. **並行處理**：多個 Claude 實例同時工作
4. **增量更新**：避免重寫整個檔案
5. **工具整合**：善用 MCP 擴展功能

## 疑難排解

### 常見問題

**Q: Claude 找不到檔案**
A: 使用絕對路徑或確認工作目錄

**Q: 權限被拒絕**
A: 檢查 `.claude/settings.json` 或使用 `/permissions`

**Q: 上下文混亂**
A: 使用 `/clear` 重置對話

**Q: 執行速度慢**
A: 分解任務、使用批次處理

## 容易被忽略的實用功能

### 隱藏寶藏功能

1. **漸進式思考模式**：從 `think` 到 `ultrathink`，根據問題複雜度選擇
2. **計畫模式 (-p)**：規劃而不執行，適合重大決策
3. **圖片理解能力**：可以分析設計稿、流程圖、錯誤截圖
4. **管道串接**：將 Claude 整合到現有工具鏈
5. **Hooks 系統**：自動化重複性檢查和處理
6. **--resume 功能**：接續中斷的對話
7. **檔案監控模式**：即時回應檔案變更
8. **子代理功能**：分解複雜任務並行處理

### 效率提升技巧

1. **批次權限設定**：使用 `.claude/settings.json` 避免重複授權
2. **自訂斜線命令**：將常用操作模板化
3. **環境變數預設**：設定專案特定的預設行為
4. **多實例協作**：同時執行多個 Claude 處理不同任務
5. **JSON 輸出模式**：便於程式化處理結果

## 總結

Claude Code 不僅是一個 AI 編程助手，更是能真正理解和參與軟體開發流程的智慧夥伴。透過合理配置和正確的使用方式，它能顯著提升開發效率，簡化複雜任務，並幫助團隊保持程式碼品質。

### 核心價值

- **低階且無偏見**：不強制工作流程，適應你的開發方式
- **代理人模式**：自主完成任務，減少人工介入
- **深度整合**：與現有工具鏈無縫配合
- **漸進式功能**：從簡單查詢到複雜系統設計都能勝任

### 最佳使用策略

1. **先探索再行動**：使用思考模式和計畫模式深入分析
2. **善用視覺輸入**：截圖、設計稿、架構圖都能理解
3. **建立專屬配置**：透過 CLAUDE.md 和設定檔客製化
4. **自動化重複工作**：使用 Headless 模式和 Hooks
5. **多模式結合**：根據任務選擇適合的模式和參數

從簡單的程式碼問答到複雜的架構重構，從個人專案到團隊協作，Claude Code 都能提供強大支援。關鍵在於理解其設計理念，掌握核心功能，並根據實際需求調整工作流程。

隨著持續使用和優化配置，Claude Code 將成為你開發工具箱中不可或缺的一部分，讓編程變得更高效、更愉快。

## 延伸資源

- [官方文件](https://docs.anthropic.com/claude-code)
- [MCP 協議規範](https://modelcontextprotocol.io)
- [社群最佳實踐](https://github.com/anthropics/claude-code-examples)
- [影片教學](https://www.youtube.com/anthropic)