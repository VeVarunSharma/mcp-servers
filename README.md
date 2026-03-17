# MCP Servers Monorepo

This repository is a monorepo containing multiple implementations of Model Context Protocol (MCP) servers in different languages and frameworks. Each subproject demonstrates how to build an MCP-compatible server using popular technologies.

It also includes **ready-to-use MCP configurations** for VS Code, GitHub Copilot, and examples of MCP governance (allowlists, registries, guardrails) that teams can adopt.

## Projects

| Server | Language | Transport | Location |
|--------|----------|-----------|----------|
| **Express MCP** | TypeScript + Express.js | HTTP (streamable) | [`mcps/mcp-with-express/`](mcps/mcp-with-express/) |
| **FastMCP Weather** | Python + FastMCP | stdio | [`mcps/mcp-with-fastmcp/weather/`](mcps/mcp-with-fastmcp/weather/) |

Both servers provide weather tools (`get-alerts`, `get-forecast`) using the National Weather Service API.

## MCP Configuration & Governance

This repo includes ready-to-use configuration examples:

| File | Purpose |
|------|---------|
| [`.vscode/mcp.json`](.vscode/mcp.json) | VS Code MCP server configuration — auto-discovered by Copilot Chat |
| [`.vscode/settings.json`](.vscode/settings.json) | VS Code workspace settings with MCP and Copilot enabled |
| [`.github/copilot-instructions.md`](.github/copilot-instructions.md) | Repository-level Copilot custom instructions |
| [`docs/mcp-setup-guide.md`](docs/mcp-setup-guide.md) | **Comprehensive guide**: MCP setup, allowlists, registries & guardrails |
| [`docs/examples/mcp-registry.json`](docs/examples/mcp-registry.json) | Example MCP registry for organization allowlists |
| [`docs/examples/mcp-server-manifest.json`](docs/examples/mcp-server-manifest.json) | Example server manifest for registry entries |

> 📖 **New to MCP governance?** Start with the [MCP Setup Guide](docs/mcp-setup-guide.md) — it explains everything from basic VS Code config to enterprise allowlist enforcement.

## Getting Started

### 1. Open in VS Code

```bash
git clone <your-repo-url>
code mcp-servers
```

VS Code automatically discovers MCP servers from `.vscode/mcp.json`. Open Copilot Chat (Agent mode) to use the tools.

### 2. Start the Express MCP Server (HTTP)

```bash
cd mcps/mcp-with-express
pnpm install
pnpm run build
pnpm start
# Server runs at http://localhost:3000/mcp
```

### 3. Use the FastMCP Weather Server (stdio)

No manual start needed — VS Code launches it automatically via `.vscode/mcp.json`.

To run manually for testing:
```bash
cd mcps/mcp-with-fastmcp/weather
uv run weather.py
```

## Structure

```
mcp-servers/
├── .vscode/
│   ├── mcp.json              # MCP server definitions for VS Code
│   └── settings.json         # Workspace settings (Copilot, MCP enabled)
├── .github/
│   ├── copilot-instructions.md  # Copilot custom instructions
│   ├── agents/               # Agent instruction files
│   ├── instructions/         # Path-scoped Copilot instructions
│   └── prompts/              # Prompt templates
├── docs/
│   ├── mcp-setup-guide.md    # Full setup & governance guide
│   └── examples/
│       ├── mcp-registry.json       # Example org allowlist registry
│       └── mcp-server-manifest.json # Example server manifest
└── mcps/
    ├── mcp-with-express/     # TypeScript + Express MCP server
    └── mcp-with-fastmcp/     # Python + FastMCP server
        └── weather/
```

## Deployment

- Each subproject can be deployed independently (e.g., to Vercel, AWS, etc.).
- See the `vercel.json` in each subproject for Vercel deployment configuration.

## License

MIT

---

For more details, see the [MCP Setup Guide](docs/mcp-setup-guide.md) and the documentation in each subproject.
