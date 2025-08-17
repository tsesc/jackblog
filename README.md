# Jack Blog

My personal blog built with Hugo and PaperMod theme.

## 🚀 Quick Start

### Local Development
```bash
hugo server
```

Visit http://localhost:1313/

### Build for Production
```bash
hugo
```

## 📝 GitHub Pages Setup

### First Time Setup
1. Go to [Settings > Pages](https://github.com/tsesc/jackblog/settings/pages)
2. Under "Build and deployment", select **Source**: `GitHub Actions`
3. Save the settings

### Automatic Deployment
After the initial setup, the blog will automatically deploy to GitHub Pages when you push to the `main` branch.

Your blog will be available at: https://tsesc.github.io/jackblog/

## 📁 Project Structure
```
.
├── content/          # Blog posts in Markdown
├── static/           # Static files (images, etc.)
├── themes/           # Hugo themes
├── hugo.toml         # Hugo configuration
└── .github/workflows # GitHub Actions for auto-deployment
```

## ✍️ Adding New Posts
```bash
hugo new posts/my-new-post.md
```

## 🎨 Theme
This blog uses [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

## 📄 License
MIT