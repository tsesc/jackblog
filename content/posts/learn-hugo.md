+++
title = 'Hugo 完整教學指南 - 從零開始建立部落格'
date = 2025-08-14T23:57:00+08:00
draft = false
tags = ['Hugo', '教學', '靜態網站', 'Markdown', 'PaperMod']
categories = ['技術筆記']
author = 'Jack'
description = '完整的 Hugo 教學，包含安裝、配置、內容管理、圖片處理、分類標籤等所有功能'
toc = true
+++

## 🚀 為什麼選擇 Hugo？

Hugo 是一個**超快速**的靜態網站生成器，它的核心理念是：
- 你只需要寫 Markdown
- Hugo 自動轉換成美觀的 HTML 網站
- 建置速度極快（毫秒級）
- 不需要資料庫，純靜態檔案

## 📚 Hugo 的工作原理

```
content/ (你的 Markdown)  →  Hugo 處理  →  public/ (生成的 HTML)
```

### 主要目錄說明

| 目錄 | 用途 | 需要編輯嗎？ |
|------|------|-------------|
| `content/` | 放置所有 Markdown 文章 | ✅ 是 |
| `static/` | 放置圖片、CSS、JS 等靜態資源 | ✅ 需要時 |
| `hugo.toml` | 網站配置檔 | ✅ 是 |
| `themes/` | 主題模板 | ❌ 通常不用 |
| `public/` | Hugo 生成的網站 | ❌ **絕對不要** |
| `archetypes/` | 文章模板 | 🔧 偶爾 |

## 🛠️ 安裝與設定

### 1. 安裝 Hugo
```bash
# macOS
brew install hugo

# Windows (使用 Chocolatey)
choco install hugo-extended

# Linux
snap install hugo
```

### 2. 創建新網站
```bash
hugo new site myblog
cd myblog
```

### 3. 安裝主題（以 PaperMod 為例）
```bash
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

### 4. 基本配置 (hugo.toml)
```toml
baseURL = 'https://yourdomain.com/'
languageCode = 'zh-tw'
title = '我的部落格'
theme = 'PaperMod'

[pagination]
pagerSize = 5

[params]
defaultTheme = 'auto'
ShowReadingTime = true
ShowShareButtons = true
ShowPostNavLinks = true
ShowBreadCrumbs = true
ShowCodeCopyButtons = true
ShowWordCount = true
ShowToc = true
TocOpen = false
```

## ✍️ 如何新增文章

### 方法 1：使用 Hugo 命令（推薦）
```bash
hugo new posts/my-new-post.md
```

### 方法 2：手動創建 Markdown
在 `content/posts/` 目錄下創建 `.md` 檔案

## 📝 Front Matter 詳解

每個 Markdown 文件開頭的 `+++...+++` 區塊稱為 Front Matter，用來設定文章屬性：

```toml
+++
title = '文章標題'
date = 2025-08-14T23:00:00+08:00
draft = false  # false 表示發布，true 表示草稿
tags = ['標籤1', '標籤2']
categories = ['分類']
author = '作者名'
description = '文章描述，用於 SEO'
toc = true  # 顯示目錄
weight = 1  # 文章排序權重
+++
```

### 重要參數說明
- **draft**: 控制文章是否發布（true = 草稿，false = 發布）
- **tags**: 文章標籤，可多個
- **categories**: 文章分類，通常一個
- **toc**: 是否顯示文章目錄

## 🖼️ 圖片管理完整指南

### 方法 1：使用 static 目錄（全站共用圖片）

```bash
# 1. 創建圖片目錄
mkdir -p static/images

# 2. 複製圖片
cp your-image.jpg static/images/

# 3. 在 Markdown 中引用
![圖片描述](/images/your-image.jpg)
```

**適用場景**：Logo、圖標、常用圖片

### 方法 2：使用 Page Bundle（文章專屬圖片）

**目錄結構**：
```
content/posts/
└── my-article/           # 文章資料夾
    ├── index.md          # 文章內容（必須命名為 index.md）
    ├── feature.jpg       # 文章專屬圖片
    └── screenshot.png    # 另一張圖片
```

**在 index.md 中引用**：
```markdown
![特色圖片](feature.jpg)
![螢幕截圖](screenshot.png)
```

**適用場景**：文章配圖、螢幕截圖、專屬資源

### 方法 3：PaperMod 封面圖片

在 Front Matter 中設定：
```toml
+++
title = '文章標題'
[cover]
image = '/images/cover.jpg'
alt = '封面圖片描述'
caption = '圖片說明文字'
relative = false
+++
```

### 圖片最佳實踐
1. **檔案命名**：使用小寫字母和連字符 `my-image.jpg`
2. **圖片優化**：壓縮圖片減少載入時間
3. **替代文字**：總是提供有意義的 alt 文字
4. **格式選擇**：照片用 JPEG，圖標用 PNG，動畫用 GIF

## 🏷️ 分類與標籤系統

### 標籤 (Tags) vs 分類 (Categories)

| 特性 | 標籤 Tags | 分類 Categories |
|------|-----------|-----------------|
| 用途 | 具體主題關鍵詞 | 文章大類別 |
| 數量 | 可多個 | 通常一個 |
| 範例 | `Hugo`, `JavaScript` | `技術筆記`, `生活分享` |

### 設置方式
```toml
+++
tags = ['Hugo', '教學', '靜態網站']
categories = ['技術筆記']
+++
```

### 自動生成的頁面
- `/tags/` - 所有標籤列表
- `/tags/hugo/` - 特定標籤的文章
- `/categories/` - 所有分類列表
- `/categories/技術筆記/` - 特定分類的文章
- `/archives/` - 歸檔頁面（時間排序）

### 分類建議
- **技術筆記**：程式開發、工具使用
- **生活分享**：日常、旅遊、美食
- **讀書心得**：書評、讀後感
- **專案紀錄**：作品集、專案文檔

## 🎨 進階功能

### 1. 程式碼高亮
使用三個反引號包圍程式碼，支援語法高亮：

```python
def hello_world():
    print("Hello, Hugo!")
```

### 2. 表格支援
```markdown
| 標題1 | 標題2 | 標題3 |
|-------|-------|-------|
| 內容1 | 內容2 | 內容3 |
```

### 3. 數學公式（需要啟用）
```markdown
$$E = mc^2$$
```

### 4. 短代碼 (Shortcodes)
Hugo 內建短代碼功能：
```markdown
{{</* youtube VIDEO_ID */>}}
{{</* x user="USERNAME" id="TWEET_ID" */>}}
{{</* highlight go */>}}
// 程式碼內容
{{</* /highlight */>}}
```

## 🚀 部署與發布

### 本地預覽
```bash
# 啟動開發伺服器
hugo server

# 包含草稿
hugo server -D

# 指定埠號
hugo server -p 8080
```

### 生成靜態網站
```bash
# 生成網站到 public/ 目錄
hugo

# 包含草稿
hugo -D
```

### 部署選項
1. **GitHub Pages**：免費靜態網站託管
2. **Netlify**：自動部署，支援 CI/CD
3. **Vercel**：快速部署，全球 CDN
4. **Cloudflare Pages**：免費且快速

## 💡 實用技巧

### 1. 草稿管理
- 開發時使用 `draft = true`
- 發布時改為 `draft = false`
- 使用 `hugo server -D` 預覽草稿

### 2. 內容組織
- 使用有意義的檔案名稱
- 保持目錄結構清晰
- 善用分類和標籤

### 3. SEO 優化
- 設定有意義的 `title` 和 `description`
- 使用適當的標題層級（H1, H2, H3）
- 為圖片添加 alt 文字

### 4. 效能優化
- 壓縮圖片大小
- 使用適當的圖片格式
- 啟用 Hugo 的內建壓縮功能

## 🔧 常見問題解決

### Q1: 文章不顯示？
- 檢查 `draft` 是否為 `false`
- 確認日期不是未來時間
- 使用 `hugo server -D` 查看草稿

### Q2: 圖片無法顯示？
- 確認路徑是否正確
- Static 目錄圖片使用 `/images/` 開頭
- Page Bundle 圖片直接使用檔名

### Q3: 中文亂碼？
- 確保檔案編碼為 UTF-8
- 設定 `languageCode = 'zh-tw'`

## 📚 學習資源

- [Hugo 官方文檔](https://gohugo.io/documentation/)
- [PaperMod 主題文檔](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [Hugo 中文社群](https://hugo.com.cn/)
- [Awesome Hugo](https://github.com/theNewDynamic/awesome-hugo)

## 🎯 總結

Hugo 讓你專注於**內容創作**，而不是網頁設計。記住核心要點：

1. 📝 只需要寫 Markdown
2. 🚀 Hugo 自動生成 HTML
3. 🎨 主題控制外觀
4. 📦 靜態檔案易於部署

**最重要的是**：你完全不需要修改 HTML、CSS 或 JavaScript，只要專心寫作就好！

---

*這是使用 Hugo + PaperMod 主題建立的第一篇文章*  
*持續更新中...*