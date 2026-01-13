# Settings (settings.json)

> Configure permissions, environment variables, hooks, and Claude Code behavior.

## Overview

The `settings.json` file configures Claude Code's behavior, permissions, and integrations. It controls what Claude can do, not what Claude knows (use CLAUDE.md for instructions).

Use settings for:
- Permission rules (allow, deny, ask)
- Environment variables
- Hook configurations
- Model overrides
- Attribution settings
- Sandbox configuration

## File location

Settings follow a precedence hierarchy from highest to lowest:

| Scope | Path | Overrides |
|-------|------|-----------|
| Managed | System directories (see below) | Cannot be overridden |
| Local | `.claude/settings.local.json` | Project, User |
| Project | `.claude/settings.json` | User |
| User | `~/.claude/settings.json` | Nothing |

Managed settings locations:
- macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`
- Linux/WSL: `/etc/claude-code/managed-settings.json`
- Windows: `C:\Program Files\ClaudeCode\managed-settings.json`

## File structure

JSON object with configuration keys:
```json
{
  "permissions": { ... },
  "env": { ... },
  "hooks": { ... },
  ...
}
```

## Permissions

The `permissions` object controls what tools Claude can use.

### Permission keys

| Key | Purpose |
|-----|---------|
| `allow` | Array of rules to allow without asking |
| `ask` | Array of rules to ask for confirmation |
| `deny` | Array of rules to block completely |
| `additionalDirectories` | Extra directories Claude can access |
| `defaultMode` | Default permission mode on startup |
| `disableBypassPermissionsMode` | Set to `"disable"` to prevent bypass mode |

### Permission rule format

Rules follow the pattern: `Tool` or `Tool(argument:pattern)`

| Pattern | Matches |
|---------|---------|
| `Read` | All Read operations |
| `Read(.env)` | Reading .env file |
| `Read(.env*)` | Reading files starting with .env |
| `Read(./secrets/**)` | Reading anything in secrets directory |
| `Bash(npm run:*)` | Bash commands starting with "npm run" |
| `WebFetch` | All web fetch operations |

Note: Bash rules use prefix matching, not regex.

### Permission modes

| Mode | Behavior |
|------|----------|
| `default` | Ask for each tool use |
| `acceptEdits` | Auto-accept file edits |
| `acceptAll` | Auto-accept all operations |
| `bypassPermissions` | Skip all permission checks (dangerous) |

## Environment variables

The `env` object sets environment variables for all sessions:
```json
{
  "env": {
    "KEY": "value"
  }
}
```

Variables are applied to every Claude Code session.

**Important:** Environment variables do NOT persist between bash commands. Options:
1. Activate environment before starting Claude Code
2. Use `CLAUDE_ENV_FILE` with SessionStart hook
3. Set in `.bashrc`/`.zshrc`

## Hooks

The `hooks` object configures automatic shell commands. See [Hooks reference](hooks.md) for full documentation.

Structure:
```json
{
  "hooks": {
    "<EventName>": [
      {
        "matcher": "<pattern>",
        "hooks": [
          {
            "type": "command",
            "command": "<shell command>"
          }
        ]
      }
    ]
  }
}
```

## Model configuration

| Key | Purpose |
|-----|---------|
| `model` | Override default model |

Model can also be set via `ANTHROPIC_MODEL` environment variable.

## Attribution

The `attribution` object customizes git commit and PR attribution:

| Key | Purpose |
|-----|---------|
| `commit` | Attribution text for commits (empty string to hide) |
| `pr` | Attribution text for pull requests (empty string to hide) |

Example:
```json
{
  "attribution": {
    "commit": "🤖 Generated with Claude Code",
    "pr": ""
  }
}
```

The deprecated `includeCoAuthoredBy` boolean is replaced by `attribution`.

## Sandbox configuration

The `sandbox` object configures bash sandboxing:

| Key | Purpose | Default |
|-----|---------|---------|
| `enabled` | Enable bash sandboxing | `false` |
| `autoAllowBashIfSandboxed` | Auto-approve bash when sandboxed | `true` |
| `excludedCommands` | Commands to run outside sandbox | `[]` |
| `allowUnsandboxedCommands` | Allow escape hatch | `true` |
| `network.allowUnixSockets` | Unix socket paths to allow | `[]` |
| `network.allowLocalBinding` | Allow localhost binding (macOS) | `false` |
| `network.httpProxyPort` | Custom HTTP proxy port | Auto |
| `network.socksProxyPort` | Custom SOCKS proxy port | Auto |
| `enableWeakerNestedSandbox` | Weaker sandbox for Docker (Linux) | `false` |

Example:
```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["docker"],
    "network": {
      "allowLocalBinding": true
    }
  }
}
```

## Other settings

| Key | Purpose |
|-----|---------|
| `apiKeyHelper` | Script to generate auth value |
| `cleanupPeriodDays` | Days before inactive sessions are deleted (default: 30) |
| `companyAnnouncements` | Messages to display at startup |
| `disableAllHooks` | Disable all hooks |
| `allowManagedHooksOnly` | Only allow managed hooks (managed settings only) |
| `statusLine` | Custom status line configuration (see below) |
| `fileSuggestion` | Custom file autocomplete command (see below) |
| `respectGitignore` | Respect .gitignore in file picker (default: true) |
| `outputStyle` | Output style adjustment |
| `forceLoginMethod` | Restrict login to `claudeai` or `console` |
| `forceLoginOrgUUID` | Auto-select organization on login |
| `language` | Claude's preferred response language |
| `alwaysThinkingEnabled` | Enable extended thinking by default |

## MCP settings

| Key | Purpose |
|-----|---------|
| `enableAllProjectMcpServers` | Auto-approve all project MCP servers |
| `enabledMcpjsonServers` | List of approved MCP servers |
| `disabledMcpjsonServers` | List of rejected MCP servers |
| `allowedMcpServers` | Allowlist for managed MCP control |
| `deniedMcpServers` | Denylist for managed MCP control |

## Plugin settings

| Key | Purpose |
|-----|---------|
| `enabledPlugins` | Object mapping plugin names to enabled status |
| `extraKnownMarketplaces` | Additional plugin marketplaces |
| `strictKnownMarketplaces` | Allowlist of permitted marketplaces (managed only) |

## AWS and cloud settings

| Key | Purpose |
|-----|---------|
| `awsAuthRefresh` | Script to refresh AWS credentials |
| `awsCredentialExport` | Script to export AWS credentials |
| `otelHeadersHelper` | Script for OpenTelemetry headers |

## File suggestion configuration

Custom autocomplete for `@file` references:

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  }
}
```

The script receives JSON via stdin:
```json
{"query": "src/comp"}
```

Output newline-separated file paths to stdout:
```
src/components/Button.tsx
src/components/Modal.tsx
src/components/Form.tsx
```

## Status line configuration

Custom status line display:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  }
}
```

The script output is displayed in the status line area.

Use `/statusline` command to configure interactively.

## Key environment variables

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API authentication |
| `ANTHROPIC_AUTH_TOKEN` | Custom auth header |
| `ANTHROPIC_MODEL` | Override default model |
| `CLAUDE_CODE_SHELL` | Override shell detection |
| `CLAUDE_CODE_USE_BEDROCK` | Use AWS Bedrock |
| `CLAUDE_CODE_USE_VERTEX` | Use Google Vertex |
| `CLAUDE_CODE_USE_FOUNDRY` | Use Microsoft Foundry |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Disable background task feature |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | Don't modify terminal title |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | Limit file read output |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | Hide account info in UI |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Override subagent model |
| `DISABLE_TELEMETRY` | Opt out of telemetry |
| `DISABLE_AUTOUPDATER` | Disable auto-updates |
| `MAX_THINKING_TOKENS` | Enable extended thinking |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | Reset dir after bash |
| `MCP_TIMEOUT` | Server startup timeout (ms) |
| `MCP_TOOL_TIMEOUT` | Tool execution timeout (ms) |
| `MAX_MCP_OUTPUT_TOKENS` | Limit MCP output size |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | Limit command expansion size |

## Settings precedence

When the same setting exists in multiple scopes:

1. Managed settings always win
2. Local overrides project and user
3. Project overrides user
4. User is the default

For arrays like permission rules, more specific scopes take precedence. A deny in project overrides an allow in user.

## Editing settings

Three ways to edit:

1. `/config` command in Claude Code opens settings interface
2. `/permissions` command for permission rules specifically
3. Edit JSON files directly

## Best practices

1. **Use project scope for team settings**: Commit `.claude/settings.json`

2. **Use local scope for personal overrides**: Keep in `.claude/settings.local.json`

3. **Be specific with permissions**: Narrow rules are safer than broad ones

4. **Deny sensitive files explicitly**: Add `.env`, secrets directories to deny list

5. **Use semantic prefixes**: For env vars, use project prefix

6. **Document in CLAUDE.md**: Explain why certain settings exist

## Common mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Overly broad allow rules | Security risk | Use specific patterns |
| Forgetting deny for secrets | Claude can read sensitive files | Add explicit deny rules |
| Wrong file location | Settings not applied | Check scope and path |
| Invalid JSON | Settings fail to load | Validate JSON syntax |
| Conflicting rules | Unexpected behavior | Check precedence hierarchy |

## Related

- [Hooks](hooks.md) - Hook configuration details
- [MCP](mcp.md) - MCP server configuration
- [CLAUDE.md](claude.md) - Instructions vs configuration
- [Plugins](plugins.md) - Plugin configuration
- [CLI](cli.md) - CLI flags and environment variables
