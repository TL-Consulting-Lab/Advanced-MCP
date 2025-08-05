# GitHub MCP Server Demo Prompts

This guide provides step-by-step instructions for setting up and using the GitHub MCP (Model Context Protocol) server in Visual Studio Code.

## 📋 Prerequisites

Before setting up the GitHub MCP server, ensure you have:

- **A GitHub account** with access to the repositories you want to work with
- **Visual Studio Code** or another MCP-compatible editor
- **Access to GitHub Copilot** - For more information, see [What is GitHub Copilot?](https://docs.github.com/en/copilot/about-github-copilot/what-is-github-copilot#getting-access-to-copilot)
- **Verified email address** on your GitHub account
- **Internet connection** for remote MCP server setup

### Additional Prerequisites for Local Setup
- **Docker** installed and running (for local MCP server setup)
- **Node.js** (if using npm package)

## 🔑 Creating Personal Access Token (PAT)

To use the GitHub MCP server with PAT authentication, you'll need to create a Personal Access Token with appropriate scopes.

### Step 1: Navigate to Token Settings
1. Go to GitHub.com and sign in to your account
2. Click your **profile picture** in the upper-right corner
3. Select **Settings** from the dropdown menu
4. In the left sidebar, click **Developer settings**
5. Under **Personal access tokens**, click **Tokens (classic)**

### Step 2: Generate New Token
1. Click **Generate new token** → **Generate new token (classic)**
2. In the **Note** field, enter a descriptive name (e.g., "MCP Server Token")
3. Set an **Expiration** date (recommended: 90 days or custom)
4. Select the required **scopes**:
   - `repo` - Full control of private repositories
   - `read:packages` - Read packages (required for local setup)
   - Additional scopes based on your needs

### Step 3: Generate and Store Token
1. Click **Generate token**
2. **Copy the token immediately** (you won't be able to see it again)
3. Store it securely (password manager recommended)

> ⚠️ **Security Warning**: Treat your PAT like a password. Never share it or commit it to code repositories.

## ⚙️ Setting Up GitHub MCP Server

You can set up the GitHub MCP server either remotely (recommended) or locally. Choose the method that best fits your needs.

### Remote Setup with PAT (Recommended)

#### Step 1: Add MCP Server in VS Code
1. Open **Command Palette**: `Ctrl+Shift+P` (Windows/Linux) or `Cmd+Shift+P` (Mac)
2. Type `mcp: add server` and press **Enter**
3. Select **HTTP (HTTP or Server-Sent Events)**
4. Enter Server URL: `https://api.githubcopilot.com/mcp/`
5. Press **Enter** to use default server ID or enter custom ID
6. Choose where to save configuration (User Settings recommended)

#### Step 2: Configure PAT Authentication
1. When prompted for OAuth, click **Cancel** to decline
2. VS Code will open the configuration file
3. Manually edit the configuration to add PAT authentication:

```json
{
    "servers": {
        "github": {
            "command": "npx",
            "args": [
                "-y",
                "@modelcontextprotocol/server-github"
            ],
            "env": {
                "GITHUB_PERSONAL_ACCESS_TOKEN": "${input:github_token}"
            },
            "type": "stdio"
        }
    },
    "inputs": [
        {
            "id": "github_token",
            "type": "promptString",
            "description": "GitHub Personal Access Token",
            "password": true
        }
    ]
}
```

#### Step 3: Restart and Authenticate
1. Click the **Restart** button in the configuration file
2. When prompted, enter your Personal Access Token
3. Press **Enter** to complete setup

### Local Setup with Docker

#### Step 1: Ensure Docker is Running
- Verify Docker is installed and running on your machine
- Test with `docker --version` in terminal

#### Step 2: Configure Local MCP Server
Add this configuration to your `.vscode/mcp.json` (for project-specific) or VS Code settings (for global):

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "github_token",
      "description": "GitHub Personal Access Token",
      "password": true
    }
  ],
  "servers": {
    "github": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GITHUB_PERSONAL_ACCESS_TOKEN",
        "ghcr.io/github/github-mcp-server"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${input:github_token}"
      }
    }
  }
}
```

## 🚀 Using GitHub MCP Server in Visual Studio Code

Once configured, you can use the GitHub MCP server to perform various GitHub operations through Copilot Chat.

### Step 1: Open Copilot Chat
1. Click the **Copilot icon** in the VS Code title bar
2. Or use Command Palette: `Ctrl+Shift+P` → `GitHub Copilot: Open Chat`

### Step 2: Switch to Agent Mode
1. In the Copilot Chat box, select **Agent** from the popup menu
2. This enables MCP server integration

### Step 3: View Available Tools
1. Click the **Select tools icon** in the Copilot Chat box
2. Under **MCP Server: GitHub**, you'll see available actions:
   - Create issues
   - List pull requests
   - Get repository information
   - Search code
   - And more...

### Step 4: Use GitHub Commands
You can now ask Copilot to perform GitHub operations. Here are some example prompts:

## 💬 Demo Prompts to Try

### Repository Information
```
Get information about the current repository
```

```
Show me the latest commits in this repository
```

```
List all branches in this repository
```

### Issues Management
```
Create a new issue titled "Fix login bug" with description "Users cannot log in with special characters in password"
```

```
List all open issues in this repository
```

```
Show me issues labeled as "bug" that are currently open
```

### Pull Requests
```
List all open pull requests in this repository
```

```
Show me the details of pull request #15
```

```
Create a pull request from branch "feature/new-login" to "main" with title "Implement new login system"
```

### Code Search and Analysis
```
Search for all functions named "getUserData" in this repository
```

```
Find all files that contain "TODO" comments
```

```
Show me the most recent changes to the README.md file
```

### Collaboration
```
List all contributors to this repository
```

```
Show me who reviewed the most recent merged pull request
```

```
Find all issues assigned to @username
```

### Advanced Operations
```
Check the CI/CD status of the latest commit
```

```
Show me the repository's security alerts
```

```
List all releases for this repository
```

## 🛠️ Troubleshooting

### Common Issues and Solutions

#### Authentication Problems
- **Issue**: "Authorization failed" or 403 errors
- **Solution**: 
  - Verify your PAT is valid and not expired
  - Ensure PAT has correct scopes (`repo`, `read:packages`)
  - Check if you're signed in to GitHub in VS Code

#### Configuration Issues
- **Issue**: MCP server not appearing in tools
- **Solution**:
  - Restart VS Code after configuration changes
  - Verify JSON syntax in configuration file
  - Check that server URL is correct

#### Agent Mode Problems
- **Issue**: Copilot not recognizing MCP commands
- **Solution**:
  - Ensure you're in Agent mode, not Ask mode
  - Verify MCP server is running and connected
  - Check VS Code output logs for errors

### General Tips
1. **Check Output Logs**: View VS Code output panel for MCP server logs
2. **Restart Services**: Try restarting VS Code or the MCP server
3. **Verify Permissions**: Ensure your PAT has necessary permissions for the action
4. **Update Token**: Create a new PAT if the current one is expired

## 📚 Additional Resources

- [Official MCP Documentation](https://modelcontextprotocol.io/introduction)
- [GitHub MCP Server Repository](https://github.com/github/github-mcp-server)
- [GitHub Copilot Documentation](https://docs.github.com/en/copilot)
- [Managing Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

## 🔒 Security Best Practices

1. **Secure Token Storage**: Use password managers for PAT storage
2. **Regular Rotation**: Rotate PATs every 90 days
3. **Minimal Scopes**: Grant only necessary permissions to tokens
4. **Monitor Usage**: Regularly review token usage in GitHub settings
5. **Revoke Unused Tokens**: Delete tokens that are no longer needed

---

*Last updated: July 2025 - Based on GitHub's official MCP server documentation*
