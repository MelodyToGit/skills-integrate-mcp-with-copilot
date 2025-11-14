# Wiki Setup Guide

This guide explains how to use the documentation files in this repository to create a GitHub Wiki for the project.

## 📚 Available Documentation

We have created **7 comprehensive documentation files** totaling over **3,000 lines** of content:

| File | Lines | Purpose | Wiki Page Name |
|------|-------|---------|----------------|
| HOME.md | 463 | Main wiki landing page | Home |
| PROJECT_OVERVIEW.md | 169 | Project introduction | Project Overview |
| PROJECT_STRUCTURE.md | 295 | File structure details | Project Structure |
| SETUP_GUIDE.md | 394 | Setup instructions | Setup Guide |
| API_REFERENCE.md | 495 | API documentation | API Reference |
| MCP_INTEGRATION.md | 579 | MCP integration guide | MCP Integration |
| README.md | 352 | Documentation index | Documentation Index |
| MCP_SYNC_GUIDE.md | 303 | ADO sync (existing, Chinese) | MCP Sync Guide |

## 🚀 Quick Setup for GitHub Wiki

### Option 1: Manual Upload

1. **Enable Wiki** in repository settings (if not already enabled)
2. **Create pages** in the Wiki section:
   - Click "Create the first page"
   - Or click "New Page" for additional pages

3. **Copy content** from each markdown file:
   - Open `docs/HOME.md`
   - Copy all content
   - Paste into Wiki "Home" page
   - Repeat for other files

4. **Suggested page structure:**
   ```
   Home (from HOME.md)
   ├── Project Overview (from PROJECT_OVERVIEW.md)
   ├── Project Structure (from PROJECT_STRUCTURE.md)
   ├── Setup Guide (from SETUP_GUIDE.md)
   ├── MCP Integration (from MCP_INTEGRATION.md)
   ├── API Reference (from API_REFERENCE.md)
   ├── Documentation Index (from README.md)
   └── MCP Sync Guide (from MCP_SYNC_GUIDE.md)
   ```

### Option 2: Clone Wiki Repository

GitHub Wikis are Git repositories. You can clone and push:

```bash
# Clone the wiki repository
git clone https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot.wiki.git

# Copy documentation files
cp docs/HOME.md skills-integrate-mcp-with-copilot.wiki/Home.md
cp docs/PROJECT_OVERVIEW.md skills-integrate-mcp-with-copilot.wiki/Project-Overview.md
cp docs/PROJECT_STRUCTURE.md skills-integrate-mcp-with-copilot.wiki/Project-Structure.md
cp docs/SETUP_GUIDE.md skills-integrate-mcp-with-copilot.wiki/Setup-Guide.md
cp docs/API_REFERENCE.md skills-integrate-mcp-with-copilot.wiki/API-Reference.md
cp docs/MCP_INTEGRATION.md skills-integrate-mcp-with-copilot.wiki/MCP-Integration.md
cp docs/README.md skills-integrate-mcp-with-copilot.wiki/Documentation-Index.md

# Commit and push
cd skills-integrate-mcp-with-copilot.wiki
git add .
git commit -m "Add comprehensive project documentation"
git push
```

### Option 3: Automated Script

Create a script to automate the process:

```bash
#!/bin/bash

WIKI_URL="https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot.wiki.git"
WIKI_DIR="wiki-temp"

# Clone wiki
git clone "$WIKI_URL" "$WIKI_DIR"

# Copy files with proper naming
cp docs/HOME.md "$WIKI_DIR/Home.md"
cp docs/PROJECT_OVERVIEW.md "$WIKI_DIR/Project-Overview.md"
cp docs/PROJECT_STRUCTURE.md "$WIKI_DIR/Project-Structure.md"
cp docs/SETUP_GUIDE.md "$WIKI_DIR/Setup-Guide.md"
cp docs/API_REFERENCE.md "$WIKI_DIR/API-Reference.md"
cp docs/MCP_INTEGRATION.md "$WIKI_DIR/MCP-Integration.md"
cp docs/README.md "$WIKI_DIR/Documentation-Index.md"
cp docs/MCP_SYNC_GUIDE.md "$WIKI_DIR/MCP-Sync-Guide.md"

# Commit and push
cd "$WIKI_DIR"
git add .
git commit -m "Add comprehensive documentation" || echo "No changes to commit"
git push

# Cleanup
cd ..
rm -rf "$WIKI_DIR"

echo "✅ Wiki updated successfully!"
```

## 📝 Wiki Page Naming Convention

GitHub Wiki converts page names to URLs. Use these names:

| Documentation File | Wiki Page Name | URL Slug |
|-------------------|----------------|----------|
| HOME.md | Home | Home |
| PROJECT_OVERVIEW.md | Project Overview | Project-Overview |
| PROJECT_STRUCTURE.md | Project Structure | Project-Structure |
| SETUP_GUIDE.md | Setup Guide | Setup-Guide |
| API_REFERENCE.md | API Reference | API-Reference |
| MCP_INTEGRATION.md | MCP Integration | MCP-Integration |
| README.md | Documentation Index | Documentation-Index |
| MCP_SYNC_GUIDE.md | MCP Sync Guide | MCP-Sync-Guide |

## 🔗 Fixing Internal Links

The documentation files use relative links that work in the repository. When moving to Wiki, update links:

### Find and Replace

**From:** `[Link Text](FILENAME.md)`  
**To:** `[Link Text](PAGENAME)`

**Examples:**
```markdown
# In repository
[Project Overview](PROJECT_OVERVIEW.md)
[Setup Guide](SETUP_GUIDE.md#section)

# In wiki
[Project Overview](Project-Overview)
[Setup Guide](Setup-Guide#section)
```

**Common replacements:**
- `PROJECT_OVERVIEW.md` → `Project-Overview`
- `PROJECT_STRUCTURE.md` → `Project-Structure`
- `SETUP_GUIDE.md` → `Setup-Guide`
- `API_REFERENCE.md` → `API-Reference`
- `MCP_INTEGRATION.md` → `MCP-Integration`
- `README.md` → `Documentation-Index`

**Keep anchor links:**
```markdown
[Setup MCP](Setup-Guide#setting-up-mcp-configuration)  # ✅ Works
```

### Automated Link Fixing Script

```bash
#!/bin/bash

# For each wiki markdown file
for file in *.md; do
  # Replace internal documentation links
  sed -i 's|(PROJECT_OVERVIEW.md)|(Project-Overview)|g' "$file"
  sed -i 's|(PROJECT_STRUCTURE.md)|(Project-Structure)|g' "$file"
  sed -i 's|(SETUP_GUIDE.md)|(Setup-Guide)|g' "$file"
  sed -i 's|(API_REFERENCE.md)|(API-Reference)|g' "$file"
  sed -i 's|(MCP_INTEGRATION.md)|(MCP-Integration)|g' "$file"
  sed -i 's|(README.md)|(Documentation-Index)|g' "$file"
  sed -i 's|(MCP_SYNC_GUIDE.md)|(MCP-Sync-Guide)|g' "$file"
  
  # Fix relative paths to repository files
  sed -i 's|(../LICENSE)|(https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/blob/main/LICENSE)|g' "$file"
  sed -i 's|(../src/)|(https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/tree/main/src/)|g' "$file"
done

echo "✅ Links fixed!"
```

## 🎨 Customizing the Wiki

### Adding a Sidebar

Create a `_Sidebar.md` file in the wiki:

```markdown
## 📚 Documentation

* **[Home](Home)**

### Getting Started
* [Project Overview](Project-Overview)
* [Setup Guide](Setup-Guide)
* [Project Structure](Project-Structure)

### Integration
* [MCP Integration](MCP-Integration)
* [MCP Sync Guide](MCP-Sync-Guide)

### Reference
* [API Reference](API-Reference)
* [Documentation Index](Documentation-Index)

---

### Quick Links
* [Repository](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot)
* [Issues](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/issues)
* [Discussions](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/discussions)
```

### Adding a Footer

Create a `_Footer.md` file:

```markdown
---

📖 **skills-integrate-mcp-with-copilot** | 
[GitHub Repo](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot) | 
[Report Issue](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/issues) | 
© 2025 GitHub Skills
```

## 🔍 Verifying the Wiki

After setup, verify:

1. **Navigation works**: All internal links function correctly
2. **Images display**: Mermaid diagrams render (GitHub Wiki supports Mermaid)
3. **Code blocks**: Syntax highlighting works
4. **Table of contents**: Auto-generated on long pages
5. **Search**: Wiki search finds relevant content

## 📊 Wiki Analytics

Track your wiki usage:
1. Go to Repository Insights
2. Check "Wiki" section
3. View page views and edits

## 🔄 Keeping Documentation Synchronized

### Manual Sync Process

1. Update files in `docs/` directory
2. Commit changes to repository
3. Copy updated files to wiki
4. Commit wiki changes

### Automated Sync (GitHub Action)

Create `.github/workflows/sync-wiki.yml`:

```yaml
name: Sync Wiki

on:
  push:
    branches: [main]
    paths:
      - 'docs/**'

jobs:
  sync-wiki:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4
      
      - name: Checkout wiki
        uses: actions/checkout@v4
        with:
          repository: ${{ github.repository }}.wiki
          path: wiki
      
      - name: Copy documentation
        run: |
          cp docs/HOME.md wiki/Home.md
          cp docs/PROJECT_OVERVIEW.md wiki/Project-Overview.md
          cp docs/PROJECT_STRUCTURE.md wiki/Project-Structure.md
          cp docs/SETUP_GUIDE.md wiki/Setup-Guide.md
          cp docs/API_REFERENCE.md wiki/API-Reference.md
          cp docs/MCP_INTEGRATION.md wiki/MCP-Integration.md
          cp docs/README.md wiki/Documentation-Index.md
      
      - name: Push to wiki
        run: |
          cd wiki
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add .
          git diff-index --quiet HEAD || git commit -m "Sync from docs/"
          git push
```

## 🎯 Best Practices

1. **Keep it simple**: Don't over-organize with too many nested pages
2. **Update regularly**: Keep wiki in sync with code changes
3. **Use clear titles**: Make pages easy to find
4. **Add examples**: Include code samples and screenshots
5. **Cross-reference**: Link related pages together
6. **Version info**: Add "Last updated" dates
7. **Test links**: Verify all internal links work

## 📱 Mobile-Friendly

GitHub Wiki pages are automatically mobile-friendly. Ensure:
- Tables are not too wide
- Code blocks are scrollable
- Images are reasonably sized

## 🌐 Access Control

Wiki inherits repository permissions:
- **Public repos**: Wiki is public
- **Private repos**: Wiki is private to collaborators
- **Organization repos**: Follow org team permissions

## ✅ Checklist for Wiki Setup

- [ ] Enable Wiki in repository settings
- [ ] Create Home page from HOME.md
- [ ] Create all documentation pages
- [ ] Fix internal links (remove .md, adjust paths)
- [ ] Add _Sidebar.md for navigation
- [ ] Add _Footer.md for footer
- [ ] Test all navigation links
- [ ] Verify images and diagrams render
- [ ] Check code syntax highlighting
- [ ] Test on mobile device
- [ ] Add to repository README link

## 🔗 Add Wiki Link to README

Update the main README.md to link to the wiki:

```markdown
## 📖 Documentation

Comprehensive documentation is available in our [Wiki](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/wiki):

- [Project Overview](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/wiki/Project-Overview)
- [Setup Guide](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/wiki/Setup-Guide)
- [MCP Integration](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/wiki/MCP-Integration)
- [API Reference](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/wiki/API-Reference)
```

## 🆘 Troubleshooting

**Problem: Images not showing**
- Use absolute URLs for images from the repository
- Example: `https://github.com/user/repo/blob/main/image.png`

**Problem: Mermaid diagrams not rendering**
- Ensure proper mermaid code block syntax:
  ````markdown
  ```mermaid
  graph TD
      A --> B
  ```
  ````

**Problem: Links broken**
- Check you removed `.md` extension
- Verify page names match exactly (case-sensitive)
- Use URL-friendly names (hyphens, not spaces)

**Problem: Can't edit wiki**
- Check you have write permissions to repository
- Verify Wiki is enabled in settings

## 📧 Support

For issues with documentation:
- Open an issue in the repository
- Tag with `documentation` label
- Reference specific documentation file

---

**Ready to set up your wiki?** Follow Option 1 or 2 above! 🚀
