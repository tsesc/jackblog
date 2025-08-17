# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Hugo-based blog using the PaperMod theme. The blog is configured for Traditional Chinese content with a focus on technical articles and personal sharing.

## Common Development Commands

```bash
# Start development server with live reload
hugo server

# Start server including draft posts
hugo server -D

# Create new blog post
hugo new posts/[post-name].md

# Build static site for production
hugo

# Build including drafts (development only)
hugo -D
```

## Content Creation Guidelines

### Creating New Posts

Always use the Hugo command to create new posts:
```bash
hugo new posts/my-article-name.md
```

### Front Matter Template

Use this comprehensive front matter for all blog posts:

```toml
+++
title = '文章標題'
date = 2025-08-17T10:00:00+08:00
draft = false  # Set to true while writing, false to publish
tags = ['標籤1', '標籤2', '標籤3']  # Multiple specific keywords
categories = ['技術筆記']  # Single main category
author = 'Jack'
description = '文章描述（用於 SEO 和社交媒體分享）'
toc = true  # Show table of contents
weight = 1  # Optional: control post ordering

# Optional: Cover image
[cover]
image = '/images/cover.jpg'  # or 'cover.jpg' for page bundle
alt = '圖片替代文字'
caption = '圖片說明'
relative = false  # true if using page bundle
+++
```

### Categories to Use

- **技術筆記**: Programming, tools, technical tutorials
- **生活分享**: Daily life, travel, food
- **讀書心得**: Book reviews, reading notes  
- **專案紀錄**: Project documentation, portfolio

### Image Management

**Option 1: Static Images (site-wide)**
```bash
# Place image in:
static/images/my-image.jpg

# Reference in markdown:
![描述](/images/my-image.jpg)
```

**Option 2: Page Bundle (post-specific)**
```
content/posts/my-article/
├── index.md       # Must be named index.md
├── image1.jpg
└── image2.png

# Reference in markdown:
![描述](image1.jpg)
```

## Project Structure

```
jackblog/
├── content/          # All markdown content
│   └── posts/       # Blog posts
├── static/          # Static assets (images, files)
│   └── images/      # Image storage
├── themes/PaperMod/ # Theme (don't modify)
├── archetypes/      # Content templates
├── public/          # Generated site (git ignored)
└── hugo.toml        # Site configuration
```

## Writing Best Practices

### Markdown Formatting

- Use descriptive headings (##, ###, ####)
- Always provide alt text for images
- Use tables for structured data
- Include code blocks with language specification

### Code Blocks
```markdown
```language
code here
```

### Shortcodes Available
- YouTube: `{{< youtube VIDEO_ID >}}`
- Tweet: `{{< tweet TWEET_ID >}}`
- Gist: `{{< gist USERNAME GIST_ID >}}`

## Configuration Notes

- Site language: Traditional Chinese (zh-tw)
- Theme: PaperMod with dark mode toggle
- Features enabled:
  - Reading time
  - Word count
  - Table of contents
  - Code copy buttons
  - Breadcrumbs
  - Post navigation

## Development Workflow

1. Create new post: `hugo new posts/article-name.md`
2. Edit with `draft = true` in front matter
3. Preview: `hugo server -D`
4. When ready: Set `draft = false`
5. Build: `hugo`
6. Deploy `public/` directory

## Important Reminders

- Never edit files in `public/` directory
- Always use lowercase with hyphens for file names
- Compress images before adding (use WebP or optimized JPEG)
- Tag posts appropriately for better discoverability
- Keep post URLs stable once published (SEO)