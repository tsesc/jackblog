# Hugo 配置指南 - hugo.toml 詳解

## 📌 最重要的設定

### 1. **基本設定**（必須修改）
```toml
baseURL = 'https://tsesc.github.io/jackblog/'  # 你的網站實際 URL
title = 'Jack Blog'                            # 網站標題
languageCode = 'zh-tw'                         # 語言設定
```

### 2. **Google Analytics 設定**
```toml
[params.analytics.google]
# 方法 1: Google Analytics 4 (推薦)
SiteVerificationTag = 'G-XXXXXXXXXX'  # 從 Google Analytics 獲取

# 方法 2: Universal Analytics (舊版)
[googleAnalytics]
ID = 'UA-XXXXXXXXX-X'
```

#### 如何獲取 Google Analytics ID：
1. 前往 [Google Analytics](https://analytics.google.com/)
2. 創建新的資源（Property）
3. 選擇「網站」平台
4. 獲取測量 ID（G-開頭）
5. 將 ID 填入配置文件

### 3. **社交媒體連結**
```toml
[[params.socialIcons]]
name = 'github'
url = 'https://github.com/tsesc'  # 修改為你的 GitHub

[[params.socialIcons]]
name = 'email'
url = 'mailto:your.email@gmail.com'  # 修改為你的 Email
```

## 🎨 外觀客製化

### 主題模式
```toml
defaultTheme = 'auto'  # 可選：auto/light/dark
disableThemeToggle = false  # true 會隱藏切換按鈕
```

### 首頁顯示模式

#### 選項 1：資訊模式（預設）
```toml
[params.profileMode]
enabled = false  # 關閉個人檔案模式

[params.homeInfoParams]
Title = "歡迎來到 Jack's Blog 👋"
Content = "你的部落格介紹"
```

#### 選項 2：個人檔案模式
```toml
[params.profileMode]
enabled = true  # 啟用個人檔案模式
title = 'Jack Tse'
subtitle = '全端工程師'
imageUrl = '/images/profile.jpg'  # 需要上傳照片到 static/images/
```

### 文章顯示設定
```toml
ShowReadingTime = true      # 顯示閱讀時間
ShowShareButtons = true     # 分享按鈕
ShowCodeCopyButtons = true  # 複製代碼按鈕
ShowWordCount = true        # 字數統計
showtoc = true             # 文章目錄
tocopen = false            # 預設展開目錄
```

## 🔍 SEO 優化

### 基本 SEO
```toml
[params]
description = "網站描述，會顯示在搜尋結果"
keywords = ['關鍵字1', '關鍵字2']
author = '你的名字'
```

### Open Graph（社交分享）
```toml
images = ["/images/site-feature.jpg"]  # 預設分享圖片
```

### 網站地圖
```toml
[sitemap]
changefreq = 'weekly'
priority = 0.5
```

## 📊 進階功能

### 評論系統

#### Disqus 評論
```toml
[params]
comments = true

[params.disqus]
shortname = 'your-disqus-shortname'
```

#### Utterances（GitHub Issues 評論）
在文章模板中添加：
```html
<script src="https://utteranc.es/client.js"
        repo="tsesc/jackblog"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

### 搜尋功能
```toml
[outputs]
home = ["HTML", "RSS", "JSON"]  # JSON 用於搜尋

[params.fuseOpts]
isCaseSensitive = false
threshold = 0.4
location = 0
distance = 1000
minMatchCharLength = 0
```

然後創建 `content/search.md`：
```markdown
+++
title = "搜尋"
layout = "search"
+++
```

### RSS 訂閱
```toml
[services.rss]
limit = 20  # RSS 文章數量

[[params.socialIcons]]
name = 'rss'
url = '/index.xml'
```

## 🛡️ 隱私與安全

### GDPR 合規設定
```toml
[privacy.googleAnalytics]
anonymizeIP = true        # 匿名化 IP
respectDoNotTrack = true  # 尊重 DNT

[privacy.youtube]
privacyEnhanced = true    # YouTube 增強隱私模式
```

## 📝 常用配置範例

### 1. 技術部落格配置
```toml
[params]
ShowCodeCopyButtons = true
showtoc = true
[markup.highlight]
lineNos = true  # 顯示行號
style = 'monokai'  # 程式碼主題
```

### 2. 個人生活部落格
```toml
[params.profileMode]
enabled = true
[params]
ShowShareButtons = true
comments = true
```

### 3. 極簡配置
```toml
[params]
hidemeta = true
ShowShareButtons = false
disableScrollToTop = true
```

## 🚀 部署前檢查清單

- [ ] `baseURL` 設定正確
- [ ] Google Analytics ID 已添加
- [ ] 社交媒體連結已更新
- [ ] Email 地址已修改
- [ ] 網站描述和關鍵字已設定
- [ ] 語言設定正確（zh-tw/en-us）
- [ ] 版權聲明已更新

## 💡 小技巧

1. **測試配置**：使用 `hugo server -D` 本地測試
2. **配置驗證**：運行 `hugo config` 查看最終配置
3. **性能優化**：啟用 `disableFingerprinting = false` 加速載入
4. **多語言**：可以設定多語言網站，參考 Hugo 文檔

## 📚 參考資源

- [Hugo 官方配置文檔](https://gohugo.io/getting-started/configuration/)
- [PaperMod 主題文檔](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [Google Analytics 設定指南](https://support.google.com/analytics/answer/9304153)

---

💡 **提示**：修改配置後記得 `git commit` 並 `git push` 來部署更新！