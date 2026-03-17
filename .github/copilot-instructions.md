# Copilot Instructions for MCP Servers Monorepo

This repository is a reference monorepo demonstrating best practices for building Model Context Protocol (MCP) servers in TypeScript/Express and Python/FastMCP.

## Repository Structure

- `mcps/mcp-with-express/` — TypeScript + Express.js MCP server (HTTP/streamable transport)
- `mcps/mcp-with-fastmcp/weather/` — Python + FastMCP MCP server (stdio transport)
- `.vscode/mcp.json` — VS Code MCP server configuration
- `docs/` — Setup guides, governance examples, and MCP registry references

## Technology Stack

| Component | Technology |
|-----------|-----------|
| Express MCP Server | TypeScript, Express.js, `@modelcontextprotocol/sdk`, Zod |
| FastMCP Server | Python 3.14+, `mcp[cli]`, `httpx` |
| Package Manager (TS) | pnpm |
| Package Manager (Py) | uv |

## Coding Standards

### TypeScript (Express Server)
- Use ES module syntax (`import`/`export`), not CommonJS.
- Use `zod` for all input/output schema validation.
- Use `async/await` for all asynchronous operations.
- Keep tool implementations focused on single responsibilities.
- Return `isError: true` in tool results for error conditions.
- Use `StreamableHTTPServerTransport` for HTTP-based MCP.

### Python (FastMCP Server)
- Follow PEP 8 style guidelines (4 spaces indentation).
- Use type hints on all function signatures — they drive MCP schema generation.
- Use `@mcp.tool()` decorators for tool registration.
- Use `httpx` for HTTP requests (async-compatible).
- Provide docstrings for all tools — they become tool descriptions in the MCP protocol.

## MCP Configuration

- VS Code MCP configs live in `.vscode/mcp.json` — commit this file for team-wide consistency.
- Never commit API keys or secrets. Use `inputs` in `mcp.json` for runtime secret prompting.
- See `docs/mcp-setup-guide.md` for governance, allowlists, and registry configuration.

## Project Conventions

- Each MCP server is self-contained under `mcps/`.
- Weather tools (`get-alerts`, `get-forecast`) use the National Weather Service (NWS) API.
- The Express server uses `/mcp` as its MCP endpoint path.
- The FastMCP server runs over stdio transport.

## Security

- Do not hardcode API keys, tokens, or credentials in source code.
- Use environment variables (`.env` files, gitignored) for local secrets.
- Use `inputs` in `.vscode/mcp.json` for runtime credential prompting.
- Only connect to trusted MCP servers — they have broad access to your workspace.
