# Project Overview

## Introduction

**skills-integrate-mcp-with-copilot** is a GitHub Skills hands-on learning exercise designed to teach developers how to expand GitHub Copilot's capabilities using the Model Context Protocol (MCP). This interactive tutorial demonstrates how to connect external services to GitHub Copilot, enabling more powerful and context-aware AI-assisted development.

## 🎯 Learning Objectives

By completing this exercise, you will learn to:

1. **Understand Model Context Protocol (MCP)** - Learn what MCP is and why it's often called "USB-C for AI"
2. **Configure MCP Servers** - Set up and connect MCP servers to GitHub Copilot
3. **Use Agent Mode** - Leverage GitHub Copilot's agent mode with MCP tools
4. **Integrate External Services** - Connect GitHub and Azure DevOps through MCP
5. **Automate Workflows** - Use Copilot to interact with issues, repositories, and work items

## 🏗️ Project Components

### 1. Sample Application

The project includes a **Mergington High School Activities Management System** - a simple FastAPI web application that demonstrates:

- **Backend API**: Python FastAPI server for managing extracurricular activities
- **Frontend**: HTML/CSS/JavaScript interface for students to view and sign up for activities
- **Data Persistence**: JSON-based storage for activities and participants
- **RESTful Endpoints**: GET and POST operations for activity management

**Application Features:**
- View all available extracurricular activities
- Sign up for activities (with capacity limits)
- Unregister from activities
- Persistent data storage

### 2. MCP Integration

The project demonstrates two MCP server integrations:

#### GitHub MCP Server
- **Type**: HTTP-based MCP server
- **URL**: `https://api.githubcopilot.com/mcp/`
- **Capabilities**: 
  - Search GitHub repositories
  - Read repository contents
  - Create and manage issues
  - Create pull requests
  - Search code across GitHub

#### Azure DevOps MCP Server
- **Type**: stdio-based MCP server
- **Package**: `@azure-devops/mcp` (npm)
- **Capabilities**:
  - Query work items
  - Create work items
  - Search work items
  - Add comments to work items
  - Batch retrieve work item details

### 3. Development Environment

The project provides a complete development environment setup:

- **Dev Container**: Pre-configured with Python 3.13, Node.js, and Docker
- **VS Code Extensions**: 
  - GitHub Copilot
  - Python & debugpy
  - ESLint
- **Port Forwarding**: Automatic forwarding of port 8000 for the web app
- **Auto-setup**: Dependencies installed via `postCreateCommand`

### 4. Exercise Workflow

The learning experience is structured through GitHub Actions workflows:

- **Step 0**: Start exercise and create initial issue
- **Step 1**: Introduction to MCP and environment setup
- **Step 2**: Using Agent Mode with GitHub MCP
- **Step 3**: Advanced MCP features
- **Step 4**: Final review and completion
- **Automated Validation**: GitHub Actions automatically validate progress

## 🎓 Learning Path

### Phase 1: Environment Setup
1. Create a GitHub Codespace
2. Verify extensions are installed
3. Run the sample application
4. Explore the codebase

### Phase 2: MCP Configuration
1. Create `.vscode/mcp.json` configuration
2. Start the GitHub MCP server
3. Authenticate with GitHub
4. Verify MCP tools are available

### Phase 3: Using MCP with Copilot
1. Enable Agent Mode in Copilot Chat
2. Search for similar projects on GitHub
3. Compare features and generate ideas
4. Create issues directly from Copilot
5. Implement features with MCP assistance

### Phase 4: Advanced Integration
1. Configure Azure DevOps MCP server
2. Sync work items between ADO and GitHub
3. Automate issue creation
4. Build custom workflows

## 🚀 Key Technologies

- **Python 3.13**: Backend programming language
- **FastAPI**: Modern web framework for building APIs
- **Uvicorn**: ASGI server for Python web apps
- **Node.js**: Runtime for MCP servers
- **GitHub Copilot**: AI-powered code completion and chat
- **Model Context Protocol**: Standardized protocol for AI tool integration
- **GitHub Actions**: CI/CD and workflow automation
- **VS Code**: Integrated development environment
- **Dev Containers**: Containerized development environment

## 📁 Repository Structure

```
skills-integrate-mcp-with-copilot/
├── .devcontainer/          # Dev container configuration
│   └── devcontainer.json   # Container setup with Python, Node.js
├── .github/                # GitHub-specific files
│   ├── steps/              # Exercise step instructions
│   └── workflows/          # Automation workflows
├── .vscode/                # VS Code configuration
│   ├── launch.json         # Debug configuration
│   └── mcp.json            # MCP server configuration
├── docs/                   # Documentation
│   └── MCP_SYNC_GUIDE.md   # Azure DevOps MCP integration guide
├── src/                    # Source code
│   ├── app.py              # FastAPI application
│   ├── activities.json     # Activity data storage
│   └── static/             # Frontend files
│       ├── index.html      # Web interface
│       ├── app.js          # Frontend logic
│       └── styles.css      # Styling
├── LICENSE                 # MIT License
├── README.md               # Welcome message
└── requirements.txt        # Python dependencies
```

## 🌟 Why This Matters

This exercise teaches critical skills for modern AI-assisted development:

1. **Reduced Context Switching**: Access multiple services from one interface
2. **Enhanced Productivity**: Let AI handle routine tasks and API interactions
3. **Standardized Integration**: Use MCP instead of custom integrations
4. **Future-Ready Skills**: MCP is becoming the standard for AI tool integration
5. **Real-World Applications**: Learn patterns applicable to production environments

## 🔗 Related Resources

- [Model Context Protocol Official Site](https://modelcontextprotocol.io/)
- [GitHub Copilot Documentation](https://docs.github.com/copilot)
- [GitHub MCP Server](https://github.com/github/github-mcp-server)
- [Azure DevOps MCP Server](https://www.npmjs.com/package/@azure-devops/mcp)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [GitHub Skills](https://learn.github.com/skills)

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](../LICENSE) file for details.

Copyright © 2025 GitHub Skills
