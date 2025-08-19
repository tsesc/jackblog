# Git Commit and Push Command

## Purpose
This command guides Claude Code to automatically commit blog post changes and push them to GitHub after content updates.

## Trigger
- When user says "commit and push" or "push to github"
- After completing blog post edits
- When user mentions "deploy" or "publish"

## Steps

### 1. Check Git Status
```bash
git status
```

### 2. Stage Changes
```bash
# Stage all changed files in content/posts/
git add content/posts/

# Stage .claude directory if updated
git add .claude/

# Stage any related images
git add static/images/

# Or stage everything
git add .
```

### 3. Create Commit
```bash
# Automatically generate commit message based on changes
git commit -m "$(cat <<'EOF'
feat: Update blog content

- Updated blog posts
- Added/modified content
- Improved formatting and structure

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### 4. Push to GitHub
```bash
# Push to main branch
git push origin main

# Or if first push with upstream
git push -u origin main
```

### 5. Verify Cloudflare Pages Deployment
After pushing to GitHub, Cloudflare Pages will automatically trigger a deployment.
Check your Cloudflare dashboard for deployment status at:
https://dash.cloudflare.com/

## Commit Message Guidelines

### For New Posts
```
feat: Add blog post about [topic]

- Created new post: [post-title]
- Added tags: [tag1, tag2, tag3]
- Category: [category-name]

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### For Updates
```
fix: Update [post-name] content

- Fixed typos and grammar
- Updated code examples
- Added new section about [topic]

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### For Multiple Changes
```
feat: Update multiple blog posts

- Updated claude-code-guide.md with new features
- Added images to [post-name]
- Fixed broken links in [post-name]

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## Important Notes

1. **Always check git status first** to understand what changes will be committed
2. **Review changes** with `git diff` before committing
3. **Use meaningful commit messages** that describe what was changed
4. **Ensure draft status** is set correctly (`draft = false` for published posts)
5. **Verify build success** before pushing (run `hugo` locally)
6. **Auto-stage .claude directory** when commands or CLAUDE.md are updated

## Common Scenarios

### Scenario 1: Publishing New Post
1. Ensure `draft = false` in front matter
2. Run `hugo` to verify build
3. Stage the new post file with `git add content/posts/`
4. Commit with descriptive message
5. Push to GitHub with `git push origin main`

### Scenario 2: Updating Existing Post
1. Make content changes
2. Update `date` field if significant update
3. Stage changes
4. Commit with update description
5. Push to GitHub

### Scenario 3: Adding Images
1. Add images to `static/images/` or page bundle
2. Reference in markdown
3. Stage both images and markdown files
4. Commit together
5. Push to GitHub

## Error Handling

If push fails:
```bash
# Pull latest changes first
git pull origin main --rebase

# Resolve any conflicts
# Then push again
git push origin main
```

If Cloudflare Pages deployment fails:
- Check the Cloudflare Pages dashboard for build logs
- Verify your Cloudflare Pages project settings
- Ensure the build command and output directory are configured correctly

## Quick Command Reference

### All-in-One Command
```bash
# Check, add, commit and push in one go
git status && \
git add . && \
git commit -m "feat: Update blog content

- Updated blog posts and content

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>" && \
git push origin main
```

### Check Deployment Status
Visit your Cloudflare Pages dashboard to monitor deployment status:
https://dash.cloudflare.com/pages