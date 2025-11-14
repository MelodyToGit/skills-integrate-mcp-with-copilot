# Welcome to the Mergington High School Management System Wiki! 🎓

> **Note**: 将此文件内容复制到 GitHub Wiki 的 Home 页面

## 📚 Wiki 目录

### 快速开始
- [项目介绍](PROJECT_INTRODUCTION) - 完整的项目介绍和功能说明
- [安装指南](#installation) - 如何设置和运行项目
- [API 文档](#api-documentation) - REST API 接口说明

### MCP 集成
- [配置 Azure DevOps MCP](#azure-devops-mcp) - ADO 集成配置
- [使用 GitHub MCP](#github-mcp) - GitHub 工具使用
- [Copilot 工作流](#copilot-workflow) - AI 辅助开发流程

### 开发指南
- [贡献指南](#contributing) - 如何为项目贡献
- [代码规范](#code-standards) - 编码标准和最佳实践
- [测试指南](#testing) - 如何编写和运行测试

## 🚀 快速开始 {#installation}

### 1. 克隆仓库
```bash
git clone https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot.git
cd skills-integrate-mcp-with-copilot
```

### 2. 安装依赖
```bash
pip install -r requirements.txt
```

### 3. 启动应用
```bash
cd src
uvicorn app:app --reload
```

### 4. 访问应用
打开浏览器访问: http://localhost:8000

## 🔧 Azure DevOps MCP 配置 {#azure-devops-mcp}

### 安装 MCP 服务器
```bash
npm install -g @azure-devops/mcp
```

### 配置文件
在 `.vscode/mcp.json` 中配置：
```json
{
  "inputs": [
    {
      "id": "ado_org",
      "type": "promptString",
      "description": "Azure DevOps organization name"
    },
    {
      "id": "ado_pat",
      "type": "promptString",
      "description": "Azure DevOps Personal Access Token",
      "password": true
    }
  ],
  "servers": {
    "ado": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@azure-devops/mcp", "${input:ado_org}"],
      "env": {
        "AZURE_DEVOPS_EXT_PAT": "${input:ado_pat}"
      }
    }
  }
}
```

### 使用示例
在 Copilot Chat 中：
```
@ado 显示我的工作项
@ado 搜索状态为 Active 的 Bug
@ado 列出所有项目
```

## 📋 API 文档 {#api-documentation}

### 端点列表

| 方法 | 端点 | 描述 |
|------|------|------|
| GET | `/` | 重定向到主页 |
| GET | `/activities` | 获取所有活动 |
| POST | `/activities/{name}/signup` | 报名参加活动 |
| DELETE | `/activities/{name}/unregister` | 取消活动报名 |

### 详细说明

#### 获取所有活动
```http
GET /activities
```

响应:
```json
{
  "Chess Club": {
    "description": "Strategic board game club",
    "schedule": "Thursdays at 3:30 PM",
    "participants": ["student@school.com"]
  }
}
```

#### 报名活动
```http
POST /activities/Chess%20Club/signup?email=student@school.com
```

成功响应:
```json
{
  "message": "Signed up student@school.com for Chess Club"
}
```

## 🤖 Copilot 工作流 {#copilot-workflow}

### 1. ADO 到 GitHub 同步
```
Azure DevOps 工作项 → MCP 同步 → GitHub Issues
```

### 2. 分配给 Copilot
```
GitHub Issue → Assign to Copilot → 自动分析和实现
```

### 3. 代码审查
```
Copilot 创建 PR → 团队审查 → 合并到主分支
```

## 🤝 贡献指南 {#contributing}

1. Fork 本仓库
2. 创建特性分支: `git checkout -b feature/AmazingFeature`
3. 提交更改: `git commit -m 'Add some AmazingFeature'`
4. 推送到分支: `git push origin feature/AmazingFeature`
5. 开启 Pull Request

## 📝 代码规范 {#code-standards}

### Python 代码
- 遵循 PEP 8 标准
- 使用类型提示
- 添加文档字符串

### JavaScript 代码
- 使用 ES6+ 语法
- 添加注释说明
- 保持代码简洁

## 📞 获取帮助

- 📧 [提交 Issue](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/issues)
- 💬 [GitHub Discussions](https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot/discussions)
- 📚 [完整文档](PROJECT_INTRODUCTION)

---

⭐ 喜欢这个项目？给我们一个 Star！

*维护者: Wang, MelodyW | 更新时间: 2025-11-14*
