# Setup Guide

This guide walks you through setting up the skills-integrate-mcp-with-copilot project in your development environment.

## Prerequisites

Before starting, ensure you have:

- ✅ A GitHub account
- ✅ Access to GitHub Codespaces (or Docker Desktop for local development)
- ✅ GitHub Copilot subscription (required for MCP integration)
- ✅ Basic familiarity with VS Code
- ✅ (Optional) Azure DevOps account for ADO integration

## Quick Start (Recommended)

### Option 1: GitHub Codespaces (Easiest)

1. **Create a Codespace**
   
   Click the button below or go to the repository and click "Code" → "Codespaces" → "Create codespace on main":
   
   [![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/MelodyToGit/skills-integrate-mcp-with-copilot?quickstart=1)

2. **Wait for Setup**
   
   The codespace will automatically:
   - Pull the dev container image (Python 3.13)
   - Install Node.js
   - Install Python dependencies (`pip install -r requirements.txt`)
   - Set up VS Code extensions (Copilot, Python, ESLint)

3. **Verify Installation**
   
   Once the codespace is ready, check the installed tools:
   
   ```bash
   python --version   # Should show Python 3.13.x
   node --version     # Should show Node.js 20.x or later
   pip list           # Should show fastapi and uvicorn
   ```

4. **Start the Application**
   
   - Open the **Run and Debug** panel (Ctrl+Shift+D)
   - Click the green ▶️ play button
   - Or press F5 to start debugging

5. **Access the Web App**
   
   - Go to the **Ports** tab (bottom panel)
   - Click the globe icon next to port 8000
   - The Mergington High School Activities page should open

✅ **You're ready to start the exercise!**

### Option 2: Local Development (Advanced)

#### Requirements

- Docker Desktop installed and running
- VS Code with Remote-Containers extension
- Git installed

#### Steps

1. **Clone the Repository**
   
   ```bash
   git clone https://github.com/MelodyToGit/skills-integrate-mcp-with-copilot.git
   cd skills-integrate-mcp-with-copilot
   ```

2. **Open in VS Code**
   
   ```bash
   code .
   ```

3. **Reopen in Container**
   
   - VS Code will detect the `.devcontainer` configuration
   - Click "Reopen in Container" in the notification
   - Or: Press F1 → "Dev Containers: Reopen in Container"

4. **Wait for Build**
   
   The container build process will:
   - Build the Python 3.13 environment
   - Install Node.js and Docker
   - Run `pip install -r requirements.txt`
   - Configure VS Code extensions

5. **Verify Setup**
   
   ```bash
   python --version
   node --version
   pip list | grep -E "(fastapi|uvicorn)"
   ```

## Detailed Configuration

### 1. Installing Python Dependencies

If dependencies are not installed automatically:

```bash
pip install -r requirements.txt
```

**Expected output:**
```
Successfully installed fastapi-0.104.1 uvicorn-0.24.0 ...
```

### 2. Setting Up MCP Configuration

**Create `.vscode/mcp.json`** (if following the exercise):

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

**For Azure DevOps integration**, add:

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
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
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

### 3. Starting MCP Servers

1. **Open `.vscode/mcp.json`**
2. Click the **Start** button that appears above the configuration
3. **Authenticate** when prompted
4. Verify the server status changes to "Running"

![MCP Start Button](https://github.com/user-attachments/assets/15a3d885-1c13-40b4-8d59-87b478ddd8a0)

### 4. Enabling Agent Mode

1. Open **Copilot Chat** panel (Ctrl+Alt+I or click Copilot icon)
2. Select **Agent** mode from the dropdown
3. Click the **🛠️ icon** to see available MCP tools

![Agent Mode Selection](https://github.com/user-attachments/assets/201e08ab-14a0-48bf-824e-ba4f8f43f8ab)

## Running the Application

### Method 1: Using VS Code Debugger

1. Press **F5** or go to Run → Start Debugging
2. The debugger will start uvicorn with hot reload
3. Application runs on `http://localhost:8000`

**Debug configuration** (`.vscode/launch.json`):
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: FastAPI",
      "type": "debugpy",
      "request": "launch",
      "module": "uvicorn",
      "args": ["src.app:app", "--reload", "--host", "0.0.0.0"],
      "jinja": true
    }
  ]
}
```

### Method 2: Command Line

```bash
cd src
python app.py
```

Or using uvicorn directly:

```bash
uvicorn src.app:app --reload --host 0.0.0.0 --port 8000
```

### Method 3: FastAPI with Hot Reload

```bash
cd /home/runner/work/skills-integrate-mcp-with-copilot/skills-integrate-mcp-with-copilot
python -m uvicorn src.app:app --reload
```

## Accessing the Application

### In Codespaces

1. Go to the **Ports** tab (bottom panel)
2. Find port 8000
3. Click the globe 🌐 icon to open in browser
4. Or click the address to copy it

### Locally

Open your browser and navigate to:
- **Application**: http://localhost:8000
- **API Docs (Swagger)**: http://localhost:8000/docs
- **Alternative Docs (ReDoc)**: http://localhost:8000/redoc

## Testing the Setup

### 1. Test the Web Interface

1. Navigate to http://localhost:8000
2. You should see the Mergington High School Activities page
3. Try viewing different activities
4. Test signing up for an activity

### 2. Test the API

**Get all activities:**
```bash
curl http://localhost:8000/activities
```

**Sign up for an activity:**
```bash
curl -X POST "http://localhost:8000/activities/Chess%20Club/signup?email=test@mergington.edu"
```

**View API documentation:**
Open http://localhost:8000/docs in your browser

### 3. Test MCP Integration

1. Open Copilot Chat
2. Ensure Agent mode is enabled
3. Try a prompt:
   ```
   Search for repositories about school management systems
   ```
4. Copilot should use the GitHub MCP tools

## Troubleshooting

### Issue: Dev Container Fails to Build

**Solution:**
- Ensure Docker is running
- Try: "Dev Containers: Rebuild Container"
- Check Docker disk space

### Issue: Python Dependencies Not Installing

**Solution:**
```bash
pip install --upgrade pip
pip install -r requirements.txt --no-cache-dir
```

### Issue: Port 8000 Already in Use

**Solution:**
```bash
# Find process using port 8000
lsof -i :8000  # On macOS/Linux
netstat -ano | findstr :8000  # On Windows

# Kill the process or use a different port
uvicorn src.app:app --reload --port 8001
```

### Issue: MCP Server Won't Start

**Check:**
1. Node.js is installed: `node --version` (need v18+)
2. Network connectivity
3. GitHub authentication is valid
4. VS Code Copilot extension is up to date (v1.296.0+)

**Restart the server:**
- Click "Stop" in mcp.json
- Wait a few seconds
- Click "Start" again

### Issue: Agent Mode Not Available

**Solutions:**
1. Update VS Code to v1.99.0 or later
2. Update Copilot extension to v1.296.0 or later
3. Check settings:
   - Open Settings (Ctrl+,)
   - Search for "agent"
   - Enable "Chat: Agent Enabled"
   - Enable "Chat: MCP Enabled"

### Issue: Activities Not Persisting

**Check:**
- `src/activities.json` file exists and is writable
- File permissions are correct
- Application has write access to the directory

**Reset data:**
```bash
cd src
git checkout activities.json
```

## Environment Variables

### For GitHub MCP
No environment variables needed - authentication is handled through VS Code.

### For Azure DevOps MCP

Set these in your environment:

```bash
export AZURE_DEVOPS_EXT_PAT="your_pat_token"
export ADO_ORG="your_org_name"
```

Or create a `.env` file (not tracked by git):
```
AZURE_DEVOPS_EXT_PAT=your_pat_token
ADO_ORG=your_org_name
```

## Next Steps

Once your setup is complete:

1. 📚 Read the [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) to understand the project
2. 🏗️ Explore [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) to learn the codebase
3. 🔌 Check [MCP_INTEGRATION.md](MCP_INTEGRATION.md) for MCP usage
4. 📖 Review [API_REFERENCE.md](API_REFERENCE.md) for API details
5. 🎓 Follow the exercise steps in `.github/steps/`

## Additional Resources

- [VS Code Remote Development](https://code.visualstudio.com/docs/remote/remote-overview)
- [GitHub Codespaces Documentation](https://docs.github.com/codespaces)
- [FastAPI Tutorial](https://fastapi.tiangolo.com/tutorial/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [GitHub Copilot Documentation](https://docs.github.com/copilot)

## Getting Help

If you encounter issues:

1. Check this troubleshooting section
2. Review the exercise step instructions
3. Check [GitHub Copilot status](https://www.githubstatus.com/)
4. Open an issue in the repository
5. Ask in GitHub Community Discussions

---

**Ready to begin?** Head over to the first exercise step in `.github/steps/1-step.md`! 🚀
