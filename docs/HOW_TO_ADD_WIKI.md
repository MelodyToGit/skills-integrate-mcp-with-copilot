# 如何在 GitHub Wiki 中添加文档

由于 GitHub Wiki 需要通过 Web 界面首次初始化，请按以下步骤操作：

## 方法 1: 通过 GitHub Web 界面（推荐）

### 步骤 1: 启用 Wiki
1. 访问仓库主页: https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot
2. 点击顶部的 **Settings** 标签
3. 在左侧菜单找到 **Features** 部分
4. 确保 **Wikis** 复选框已勾选

### 步骤 2: 创建 Wiki 首页
1. 点击仓库顶部的 **Wiki** 标签
2. 点击 **Create the first page** 按钮
3. 将 `docs/WIKI_HOME.md` 的内容复制粘贴到编辑器中
4. 点击 **Save Page** 保存

### 步骤 3: 添加项目介绍页面
1. 在 Wiki 页面，点击 **New Page** 按钮
2. 页面标题输入: `PROJECT_INTRODUCTION`
3. 将 `docs/PROJECT_INTRODUCTION.md` 的内容复制粘贴进去
4. 点击 **Save Page** 保存

## 方法 2: 通过 Git 命令行

### 步骤 1: 首先在 Web 界面创建第一个页面
按照方法 1 的步骤 1-2，先创建 Wiki 首页

### 步骤 2: 克隆 Wiki 仓库
```bash
cd ~/Documents
git clone https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot.wiki.git
cd skills-integrate-mcp-with-copilot.wiki
```

### 步骤 3: 复制文档
```bash
# 从项目仓库复制文档
cp ../skills-integrate-mcp-with-copilot/docs/WIKI_HOME.md ./Home.md
cp ../skills-integrate-mcp-with-copilot/docs/PROJECT_INTRODUCTION.md ./PROJECT-INTRODUCTION.md
```

### 步骤 4: 提交并推送
```bash
git add .
git commit -m "Add project documentation to Wiki"
git push
```

## 已创建的文档文件

项目中已包含以下文档：

1. **docs/PROJECT_INTRODUCTION.md**
   - 完整的项目介绍文档
   - 包含技术架构、API 文档、使用指南等
   - 适合作为 Wiki 的主要内容页面

2. **docs/WIKI_HOME.md**
   - Wiki 首页内容
   - 包含快速导航和常用链接
   - 应该作为 Wiki 的 Home 页面

## 推荐的 Wiki 结构

```
Wiki 首页 (Home)
├── 项目介绍 (PROJECT_INTRODUCTION)
├── 安装指南 (Installation-Guide)
├── API 文档 (API-Documentation)
├── MCP 集成配置 (MCP-Configuration)
│   ├── Azure DevOps MCP
│   └── GitHub MCP
├── 开发指南 (Development-Guide)
│   ├── 代码规范
│   ├── 测试指南
│   └── 贡献指南
└── 常见问题 (FAQ)
```

## 下一步

1. ✅ 文档已创建在 `docs/` 目录
2. ⏳ 访问 GitHub 仓库启用 Wiki
3. ⏳ 将文档内容添加到 Wiki
4. ⏳ 根据需要添加更多页面

## 参考链接

- [GitHub Wiki 文档](https://docs.github.com/en/communities/documenting-your-project-with-wikis)
- [Markdown 语法指南](https://guides.github.com/features/mastering-markdown/)
- [项目仓库](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot)

---

*如有问题，请在仓库中提交 Issue*
