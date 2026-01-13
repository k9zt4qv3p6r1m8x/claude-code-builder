# MCP (Model Context Protocol)

> Connect Claude Code to external tools and data sources through standardized integrations.

## Overview

MCP (Model Context Protocol) is an open standard for AI-tool integrations. MCP servers give Claude Code access to external tools, databases, APIs, and services.

Use MCP for:
- Connecting to databases
- Integrating with issue trackers (GitHub, Jira, Linear)
- Accessing monitoring tools (Sentry, Datadog)
- Browser automation (Puppeteer)
- Custom API integrations
- Any external tool or data source

MCP provides tools to Claude. Skills teach Claude how to use tools.

## File location

| Scope | Path | Applies to |
|-------|------|------------|
| Project | `.mcp.json` | Anyone in this repository (shared via version control) |
| Local | `~/.claude.json` (under project path) | You only, in this project |
| User | `~/.claude.json` | You, across all projects |

Project scope is stored in `.mcp.json` at project root and can be committed to version control.

## Transport types

MCP servers communicate via three transport types:

| Transport | Use for | Connection |
|-----------|---------|------------|
| `http` | Remote cloud services | HTTP endpoint URL |
| `sse` | Remote services (deprecated) | Server-Sent Events URL |
| `stdio` | Local processes | Command that runs on your machine |

HTTP is recommended for remote servers. Stdio is used for local tools.

## Adding servers via CLI

### Remote HTTP server
```
claude mcp add --transport http <name> <url>
```

Add headers for authentication:
```
claude mcp add --transport http <name> <url> --header "Authorization: Bearer <token>"
```

### Remote SSE server (deprecated)
```
claude mcp add --transport sse <name> <url>
```

### Local stdio server
```
claude mcp add --transport stdio <name> -- <command> [args...]
```

Add environment variables:
```
claude mcp add --transport stdio --env KEY=value <name> -- <command> [args...]
```

**Important:** All options (`--transport`, `--env`, `--scope`, `--header`) must come before the server name. Use `--` to separate the server name from the command.

**Windows Note:** On native Windows (not WSL), use `cmd /c` wrapper for `npx` commands:
```
claude mcp add --transport stdio my-server -- cmd /c npx -y @scope/server
```

## Scope flag

Use `--scope` to specify where the configuration is stored:

| Scope | Flag | Stored in | Shared |
|-------|------|-----------|--------|
| Local | `--scope local` (default) | `~/.claude.json` under project | No |
| Project | `--scope project` | `.mcp.json` | Yes (via version control) |
| User | `--scope user` | `~/.claude.json` | No |

## Managing servers

| Command | Purpose |
|---------|---------|
| `claude mcp list` | List all configured servers |
| `claude mcp get <name>` | Get details for a server |
| `claude mcp remove <name>` | Remove a server |
| `/mcp` | Check status within Claude Code |

## Configuration file structure

The `.mcp.json` file at project root:
```json
{
  "mcpServers": {
    "<server-name>": {
      "command": "<command>",
      "args": [],
      "env": {}
    }
  }
}
```

For HTTP servers:
```json
{
  "mcpServers": {
    "<server-name>": {
      "type": "http",
      "url": "<url>",
      "headers": {}
    }
  }
}
```

## Environment variable expansion

Environment variables can be expanded in `.mcp.json` files:

| Syntax | Behavior |
|--------|----------|
| `${VAR}` | Expands to value of VAR |
| `${VAR:-default}` | Expands to VAR if set, otherwise uses default |

Expansion works in: `command`, `args`, `env`, `url`, `headers`.

## Authentication

Many cloud MCP servers require OAuth authentication.

1. Add the server with `claude mcp add`
2. Run `/mcp` within Claude Code
3. Follow browser prompts to authenticate

Tokens are stored securely and refreshed automatically.

Use "Clear authentication" in `/mcp` menu to revoke access.

## Dynamic tool updates

MCP servers can notify Claude Code when tools change via `list_changed` notifications. This allows servers to dynamically add, remove, or modify tools without restarting.

## MCP resources

MCP servers can expose resources that you reference with `@` mentions.

Format: `@server:protocol://resource/path`

Resources are fetched and included as attachments when referenced.

## MCP prompts as commands

MCP servers can expose prompts that become slash commands.

Format: `/mcp__<server-name>__<prompt-name> [arguments]`

Prompts are dynamically discovered from connected servers.

## Using Claude Code as MCP server

Claude Code can act as an MCP server for other applications:
```
claude mcp serve
```

This exposes Claude's tools (View, Edit, LS, etc.) to MCP clients.

## Output limits

MCP tool outputs are limited to prevent context overflow.

| Setting | Default |
|---------|---------|
| Warning threshold | 10,000 tokens |
| Maximum output | 25,000 tokens |

Set `MAX_MCP_OUTPUT_TOKENS` environment variable to adjust the limit.

## Timeout configuration

| Variable | Purpose |
|----------|---------|
| `MCP_TIMEOUT` | Server startup timeout (milliseconds) |
| `MCP_TOOL_TIMEOUT` | Tool execution timeout (milliseconds) |

## Plugin MCP servers

Plugins can bundle MCP servers that start automatically when the plugin is enabled.

Plugin servers are defined in `.mcp.json` at the plugin root or inline in `plugin.json`.

Use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths:

```json
{
  "mcpServers": {
    "plugin-tool": {
      "command": "${CLAUDE_PLUGIN_ROOT}/bin/server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  }
}
```

## Managed configuration

For organizations, MCP can be controlled via managed settings:

| File | Location |
|------|----------|
| macOS | `/Library/Application Support/ClaudeCode/managed-mcp.json` |
| Linux/WSL | `/etc/claude-code/managed-mcp.json` |
| Windows | `C:\Program Files\ClaudeCode\managed-mcp.json` |

Managed MCP takes exclusive control. Users cannot add or modify servers.

## Allowlists and denylists

In managed settings, use `allowedMcpServers` and `deniedMcpServers` to control which servers users can configure.

Restriction options:
- `serverName`: Match by configured server name
- `serverCommand`: Match exact command and arguments (stdio)
- `serverUrl`: Match URL pattern with wildcards (remote)

URL pattern matching supports wildcards (`*`). The `ref` and `path` fields require exact matches.

Example configurations:
```json
{
  "allowedMcpServers": [
    {"serverName": "approved-server"},
    {"serverUrl": "https://api.example.com/*"},
    {"serverCommand": ["npx", "-y", "@approved/mcp-server"]}
  ],
  "deniedMcpServers": [
    {"serverName": "blocked-*"},
    {"serverUrl": "http://*"}
  ]
}
```

Denylist takes precedence over allowlist.

## Best practices

1. **Use project scope for team servers**: Commit `.mcp.json` so everyone has access

2. **Use local scope for personal servers**: Keep credentials in `~/.claude.json`

3. **Set appropriate timeouts**: Increase for slow servers

4. **Handle large outputs**: Set `MAX_MCP_OUTPUT_TOKENS` for data-heavy tools

5. **Use semantic naming**: Name servers by function, not implementation

6. **Document required env vars**: Note what environment variables servers need

## Common mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Options after server name | CLI parsing fails | Put all options before server name |
| Missing `--` for stdio | Command arguments misinterpreted | Use `--` before command |
| Wrong scope | Server not found or not shared | Use appropriate `--scope` flag |
| Missing env vars | Server fails to start | Add required `--env` flags |
| Timeout too short | Server disconnects | Increase `MCP_TIMEOUT` |
| npx on Windows | Command fails | Use `cmd /c npx` wrapper |

## Importing from Claude Desktop

If you have MCP servers configured in Claude Desktop:
```
claude mcp add-from-claude-desktop
```

Select which servers to import. Works on macOS and WSL only.

## Adding from JSON

Add a server from JSON configuration:
```
claude mcp add-json <name> '<json>'
```

The JSON must conform to the MCP server configuration schema.

## Related

- [Settings](settings.md) - MCP settings and managed configuration
- [Skills](skills.md) - Teach Claude how to use MCP tools
- [Hooks](hooks.md) - Automate actions around MCP tool usage
- [Plugins](plugins.md) - Bundle MCP servers in plugins
- [CLI](cli.md) - MCP CLI commands
