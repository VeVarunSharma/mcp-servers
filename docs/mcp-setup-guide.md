# MCP Setup Guide: Configuration, Governance & Allowlists

This guide shows how to configure MCP (Model Context Protocol) servers in VS Code, GitHub Copilot, and Claude Desktop — plus how organizations can use registries, allowlists, and guardrails to govern MCP server usage across teams.

---

## Table of Contents

- [1. Quick Start](#1-quick-start)
- [2. VS Code Configuration (`.vscode/mcp.json`)](#2-vs-code-configuration-vscodemcpjson)
- [3. Copilot Custom Instructions (`.github/copilot-instructions.md`)](#3-copilot-custom-instructions-githubcopilot-instructionsmd)
- [4. Claude Desktop Configuration](#4-claude-desktop-configuration)
- [5. MCP Allowlist & Registry (Governance)](#5-mcp-allowlist--registry-governance)
- [6. Guardrails & Best Practices](#6-guardrails--best-practices)
- [7. Troubleshooting](#7-troubleshooting)

---

## 1. Quick Start

### Prerequisites
- **VS Code** 1.99+ with GitHub Copilot extension
- **Node.js** 24+ and **pnpm** (for the Express server)
- **Python** 3.14+ and **uv** (for the FastMCP server)

### Steps

1. **Clone this repo** and open it in VS Code:
   ```bash
   git clone <your-repo-url>
   code mcp-servers
   ```

2. **VS Code automatically discovers** MCP servers from `.vscode/mcp.json`. You should see a notification or can check via the MCP panel in Copilot Chat.

3. **Start the Express MCP server** (HTTP transport):
   ```bash
   cd mcps/mcp-with-express
   pnpm install
   pnpm run build
   pnpm start
   ```
   The server runs at `http://localhost:3000/mcp` by default.

4. **The FastMCP server** (stdio transport) is launched automatically by VS Code when needed — no manual start required.

5. **Open Copilot Chat** (Agent mode) and use the MCP tools:
   ```
   @weather What are the weather alerts for California?
   ```

---

## 2. VS Code Configuration (`.vscode/mcp.json`)

The `.vscode/mcp.json` file tells VS Code which MCP servers are available in your workspace. This repo includes a pre-configured file.

### File Location & Scope

| Scope | File | Applies To |
|-------|------|-----------|
| **Workspace** | `.vscode/mcp.json` | This project only |
| **User** | `~/.vscode/mcp.json` | All VS Code projects |

### Configuration Format

```jsonc
{
  "inputs": [
    {
      "id": "my-api-key",
      "description": "API key for the service",
      "type": "promptString"
    }
  ],
  "servers": {
    "server-name": {
      "type": "stdio" | "http",
      "command": "node",              // for stdio servers
      "args": ["server.js"],          // for stdio servers
      "url": "http://localhost:3000", // for http servers
      "env": {                        // optional environment variables
        "API_KEY": "${input:my-api-key}"
      }
    }
  }
}
```

### Server Types Explained

| Type | When to Use | How It Works |
|------|-------------|-------------|
| `stdio` | Local tools, CLI-based servers | VS Code launches the server as a child process and communicates via stdin/stdout |
| `http` | Remote/cloud servers, shared team servers | VS Code connects to a running HTTP endpoint using streamable HTTP transport |

### This Repo's Servers

#### Express MCP Server (HTTP)
```jsonc
"express-mcp-server": {
  "type": "http",
  "url": "${input:express-mcp-url}"  // Prompted at runtime — never hardcode URLs with credentials
}
```
- **Start the server** before connecting: `cd mcps/mcp-with-express && pnpm start`
- Deployed instances (e.g., on Vercel) work too: `https://your-app.vercel.app/mcp`

#### FastMCP Weather Server (stdio)
```jsonc
"fastmcp-weather-server": {
  "type": "stdio",
  "command": "uv",
  "args": ["run", "--directory", "${workspaceFolder}/mcps/mcp-with-fastmcp/weather", "weather.py"]
}
```
- **No manual start needed** — VS Code launches the process on demand.
- Requires `uv` and Python 3.14+ installed locally.

### Key Rules

> ⚠️ **Never commit secrets.** Use `"inputs"` to prompt for API keys at runtime.
>
> ✅ **Commit `.vscode/mcp.json`** to your repo for team-wide consistency. It's safe as long as you don't include secrets.

---

## 3. Copilot Custom Instructions (`.github/copilot-instructions.md`)

The `.github/copilot-instructions.md` file provides repository-level context to GitHub Copilot, improving the quality and consistency of code suggestions and chat responses.

### What It Does

- Copilot reads this file automatically when working in your repo.
- It guides Copilot on your tech stack, coding conventions, and project structure.
- It applies to both Copilot code completions and Copilot Chat.

### How to Enable

In VS Code settings (`.vscode/settings.json`):
```jsonc
{
  "github.copilot.chat.codeGeneration.useInstructionFiles": true
}
```

This is already configured in this repo's `.vscode/settings.json`.

### Path-Specific Instructions

For more granular control, you can create path-scoped instruction files:

```
.github/
  instructions/
    python.instructions.md         # Applies to all Python files
    typescript.instructions.md     # Applies to all TypeScript files
```

Each file uses YAML frontmatter to specify which files it applies to:

```markdown
---
applyTo: '**/*.py'
description: 'Python coding conventions'
---

# Python Instructions
- Follow PEP 8
- Use type hints on all functions
```

This repo already has several instruction files in `.github/instructions/`.

---

## 4. Claude Desktop Configuration

Claude Desktop uses a different config file format. Add this to your Claude Desktop config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```jsonc
{
  "mcpServers": {
    "fastmcp-weather": {
      "command": "uv",
      "args": [
        "run",
        "--directory",
        "/absolute/path/to/mcps/mcp-with-fastmcp/weather",
        "weather.py"
      ]
    }
  }
}
```

> Note: Claude Desktop only supports `stdio` transport. For the Express server (HTTP transport), use the MCP Inspector or VS Code.

---

## 5. MCP Allowlist & Registry (Governance)

Organizations need to control which MCP servers developers can use. GitHub Copilot supports this through **MCP registries** and **allowlist policies**.

### How It Works

```
┌─────────────────────────────────────────────────┐
│              Organization Admin                  │
│                                                  │
│  1. Creates an MCP Registry (JSON catalog)       │
│  2. Uploads registry URL to GitHub Copilot       │
│     policy settings (org or enterprise level)    │
│  3. Sets allowlist policy:                       │
│     • "Allow all" — any server, registry         │
│       entries shown as recommended               │
│     • "Registry only" — only approved servers    │
│       can be used; all others are blocked        │
│                                                  │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│              Developer's IDE                     │
│                                                  │
│  • VS Code checks the registry at runtime        │
│  • Non-registry servers are blocked (if policy   │
│    is "registry only")                           │
│  • Clear warning messages shown for blocked      │
│    servers                                       │
│  • Policy changes take effect immediately        │
│                                                  │
└─────────────────────────────────────────────────┘
```

### Example MCP Registry

An MCP registry is a JSON API that lists approved servers. See [`docs/examples/mcp-registry.json`](examples/mcp-registry.json) for a full example.

The registry format follows the MCP v0.1 spec:

```jsonc
{
  "$schema": "https://static.modelcontextprotocol.io/schemas/2025-12-11/registry.schema.json",
  "servers": [
    {
      "name": "org.example/weather-mcp",
      "title": "Weather MCP Server",
      "versions": [
        {
          "version": "1.0.0",
          "manifestUrl": "https://registry.example.com/v0.1/servers/org.example/weather-mcp/versions/1.0.0/index.json"
        }
      ]
    }
  ]
}
```

### Setting Up an MCP Allowlist

#### Step 1: Create or Host a Registry

You can:
- **Self-host** a static JSON file on any web server or GitHub Pages
- **Use Azure API Center** for a managed registry with built-in governance
- **Fork an existing registry** (e.g., [modelcontextprotocol/registry](https://github.com/modelcontextprotocol/registry))

#### Step 2: Configure the Policy in GitHub

1. Go to your **GitHub Organization** → **Settings** → **Copilot** → **Policies**
2. Under **MCP server access**, upload your registry URL
3. Choose your allowlist policy:
   - **Allow all**: Any MCP server can be used (registry entries shown as recommended)
   - **Registry only**: Only servers in the registry are allowed at runtime

#### Step 3: Verify Enforcement

Developers will see:
- ✅ **Approved servers**: Work normally with a "Recommended" badge
- ❌ **Blocked servers**: Show a clear warning and cannot be used

### Hierarchical Policy Resolution

| Level | Behavior |
|-------|----------|
| **Enterprise** | Overrides all organization policies |
| **Organization** | Applies to all repos in the org |
| **Repository** | `.vscode/mcp.json` defines available servers (subject to org policy) |

> Enterprise policies always win. If an org sets "Registry only" but the enterprise sets "Allow all", the enterprise policy applies.

---

## 6. Guardrails & Best Practices

### Security Guardrails

| Guardrail | How to Implement |
|-----------|-----------------|
| **No hardcoded secrets** | Use `inputs` in `mcp.json` for runtime prompting |
| **Restrict MCP servers** | Use org-level MCP registry + "Registry only" policy |
| **Audit MCP usage** | Monitor Copilot usage logs in GitHub org settings |
| **Pin server versions** | Use specific versions in registry manifests, not `latest` |
| **Review server code** | Only approve servers with audited source code into your registry |

### Quality Guardrails

| Guardrail | How to Implement |
|-----------|-----------------|
| **Consistent coding style** | Use `.github/copilot-instructions.md` for repo-wide conventions |
| **Path-specific rules** | Use `.github/instructions/*.instructions.md` for targeted guidance |
| **Approved patterns only** | Include approved patterns and anti-patterns in instruction files |
| **Custom chat modes** | Create `.github/chat-modes/*.chatmode.md` for specialized workflows |

### Governance Checklist for Teams

- [ ] Create and host an MCP registry listing approved servers
- [ ] Set the org-level allowlist policy to "Registry only"
- [ ] Commit `.vscode/mcp.json` to repos with approved server configs
- [ ] Create `.github/copilot-instructions.md` with team conventions
- [ ] Add path-scoped instructions for language/framework-specific rules
- [ ] Document which MCP servers are approved and why
- [ ] Review and update the registry quarterly
- [ ] Audit Copilot usage logs for unauthorized MCP server attempts

---

## 7. Troubleshooting

### MCP server not showing up in VS Code

1. **Check VS Code version**: MCP support requires VS Code 1.99+
2. **Verify `mcp.json` syntax**: Open the file and check for JSON errors (VS Code highlights them)
3. **Check the MCP panel**: Open Copilot Chat → look for the MCP tools icon (🔧)
4. **Restart VS Code**: Some changes require a window reload (`Ctrl/Cmd + Shift + P` → "Reload Window")

### "Server blocked by organization policy"

Your org admin has set a "Registry only" allowlist policy and the server isn't in the approved registry. Contact your admin to:
1. Add the server to the org's MCP registry
2. Or change the policy to "Allow all" (not recommended for production)

### stdio server fails to start

1. **Check the command exists**: Run the `command` (e.g., `uv`, `node`) in your terminal
2. **Check the path**: Ensure `args` paths are correct relative to your workspace
3. **Check dependencies**: For Python servers, ensure `uv` and required packages are installed
4. **Check logs**: VS Code Output panel → select "MCP" from the dropdown

### HTTP server connection refused

1. **Is the server running?** Start it manually first (e.g., `pnpm start`)
2. **Correct URL?** Ensure the URL includes the MCP path (e.g., `http://localhost:3000/mcp`)
3. **Firewall?** Check that the port isn't blocked
4. **CORS?** For remote servers, ensure CORS headers allow your VS Code origin

---

## Further Reading

- [VS Code MCP Configuration Reference](https://code.visualstudio.com/docs/copilot/reference/mcp-configuration)
- [GitHub Docs: Configure MCP Server Access](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-mcp-usage/configure-mcp-server-access)
- [GitHub Docs: Configure an MCP Registry](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-mcp-usage/configure-mcp-registry)
- [GitHub Docs: MCP Allowlist Enforcement](https://docs.github.com/en/copilot/reference/mcp-allowlist-enforcement)
- [MCP Registry Specification](https://github.com/modelcontextprotocol/registry)
- [Model Context Protocol Documentation](https://modelcontextprotocol.io)
