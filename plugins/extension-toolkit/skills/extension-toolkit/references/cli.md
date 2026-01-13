# CLI Reference

> Command-line interface for Claude Code with flags, subcommands, and usage patterns.

## Overview

Claude Code provides a powerful CLI for interactive sessions, scripted automation, and configuration management. Commands can be run interactively or in print mode for pipelines.

Use the CLI for:
- Starting interactive coding sessions
- Running one-off queries in scripts
- Managing MCP servers
- Configuring sessions with specific models and tools
- Piping content for analysis

## Basic commands

| Command | Description |
|---------|-------------|
| `claude` | Start interactive REPL |
| `claude "query"` | Start REPL with initial prompt |
| `claude -p "query"` | Execute query and exit (print mode) |
| `claude -c` | Continue most recent conversation |
| `claude -r "session" "query"` | Resume specific session by ID or name |
| `claude update` | Update to latest version |
| `claude doctor` | Verify installation and diagnose issues |
| `claude mcp` | Configure MCP servers |

## Piping content

Process piped content with print mode:
```bash
cat file.txt | claude -p "explain this"
git diff | claude -p "review these changes"
tail -f app.log | claude -p "alert me on errors"
```

## Session management flags

| Flag | Short | Description |
|------|-------|-------------|
| `--continue` | `-c` | Load recent conversation in current directory |
| `--resume` | `-r` | Resume specific session by ID or name |
| `--session-id` | | Use specific UUID for conversation |
| `--fork-session` | | Create new ID instead of reusing original |

## Model and behavior flags

| Flag | Description |
|------|-------------|
| `--model` | Set model: `sonnet`, `opus`, `haiku`, or full name |
| `--fallback-model` | Enable auto-fallback when primary overloaded (print mode) |
| `--agent` | Specify agent for current session |
| `--max-turns` | Limit agentic turns (print mode only) |

Model aliases:
- `sonnet` → `claude-sonnet-4-5-20250929`
- `opus` → `claude-opus-4-5-20251101`
- `haiku` → `claude-haiku-3-5-20240307`

## System prompt flags

| Flag | Behavior | Modes |
|------|----------|-------|
| `--system-prompt` | Replace entire default prompt | Interactive + Print |
| `--system-prompt-file` | Replace with file content | Print only |
| `--append-system-prompt` | Add to default prompt | Interactive + Print |

`--system-prompt` and `--system-prompt-file` are mutually exclusive.

Use `--append-system-prompt` to add requirements while keeping defaults (recommended for most cases).

## Input/Output flags

| Flag | Description |
|------|-------------|
| `--print`, `-p` | Print response without interactive mode |
| `--output-format` | Output format: `text`, `json`, `stream-json` |
| `--input-format` | Input format: `text`, `stream-json` |
| `--include-partial-messages` | Include streaming events (requires `stream-json`) |
| `--verbose` | Enable detailed logging, show full turn-by-turn output |

## Tools and permissions flags

| Flag | Description |
|------|-------------|
| `--tools` | Limit available tools (e.g., `"Bash,Edit,Read"` or `"default"` or `""` to disable all) |
| `--allowedTools` | Tools that run without permission prompt |
| `--disallowedTools` | Remove tools from model context |
| `--dangerously-skip-permissions` | Skip permission prompts (use with caution) |
| `--permission-mode` | Start in specific mode: `plan`, `default`, `acceptEdits`, `bypassPermissions` |
| `--permission-prompt-tool` | Specify MCP tool for permission handling |

## Configuration flags

| Flag | Description |
|------|-------------|
| `--settings` | Load JSON settings file or string |
| `--setting-sources` | Comma-separated sources: `user,project,local` |
| `--plugin-dir` | Load plugins from directory (repeatable) |
| `--mcp-config` | Load MCP servers from JSON file or strings |
| `--strict-mcp-config` | Use only specified MCP servers |

## Feature flags

| Flag | Description |
|------|-------------|
| `--chrome` | Enable Chrome browser integration |
| `--no-chrome` | Disable Chrome integration for this session |
| `--ide` | Auto-connect to available IDE |
| `--add-dir` | Add working directories for Claude access |

## Advanced flags

| Flag | Description |
|------|-------------|
| `--debug` | Enable debug mode with optional category filter |
| `--betas` | Include beta API headers (API key users only) |
| `--json-schema` | Get JSON validated against schema (print mode) |
| `--version`, `-v` | Show version number |

Debug categories: `"api,mcp"` or exclude with `"!statsig,!file"`

## Subagents via CLI

Define subagents inline with `--agents` flag:

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  },
  "debugger": {
    "description": "Debugging specialist for errors and test failures.",
    "prompt": "You are an expert debugger. Analyze errors, identify root causes, and provide fixes."
  }
}'
```

Fields:
- `description` (required): When to invoke the subagent
- `prompt` (required): System prompt for behavior
- `tools` (optional): Specific tools; inherits all if omitted
- `model` (optional): `sonnet`, `opus`, `haiku`

## MCP subcommands

| Command | Description |
|---------|-------------|
| `claude mcp list` | List all configured servers |
| `claude mcp get <name>` | Get details for a server |
| `claude mcp add` | Add a new MCP server |
| `claude mcp remove <name>` | Remove a server |
| `claude mcp add-from-claude-desktop` | Import from Claude Desktop |
| `claude mcp add-json <name> '<json>'` | Add from JSON configuration |
| `claude mcp serve` | Run Claude Code as MCP server |

### Adding MCP servers

HTTP server:
```bash
claude mcp add --transport http <name> <url>
claude mcp add --transport http <name> <url> --header "Authorization: Bearer <token>"
```

Stdio server:
```bash
claude mcp add --transport stdio <name> -- <command> [args...]
claude mcp add --transport stdio --env KEY=value <name> -- <command>
```

Options must come before server name. Use `--` to separate command.

Scope flag:
- `--scope local` (default): `~/.claude.json` under project
- `--scope project`: `.mcp.json` (shared)
- `--scope user`: `~/.claude.json`

## Environment variables

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API authentication |
| `ANTHROPIC_AUTH_TOKEN` | Custom auth header |
| `ANTHROPIC_MODEL` | Override default model |
| `CLAUDE_CODE_SHELL` | Override shell detection |
| `CLAUDE_CODE_USE_BEDROCK` | Use AWS Bedrock |
| `CLAUDE_CODE_USE_VERTEX` | Use Google Vertex |
| `CLAUDE_CODE_USE_FOUNDRY` | Use Microsoft Foundry |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Override subagent model |
| `DISABLE_TELEMETRY` | Opt out of telemetry |
| `DISABLE_AUTOUPDATER` | Disable auto-updates |
| `MAX_THINKING_TOKENS` | Enable extended thinking |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | Reset dir after bash |
| `MCP_TIMEOUT` | Server startup timeout (ms) |
| `MCP_TOOL_TIMEOUT` | Tool execution timeout (ms) |
| `MAX_MCP_OUTPUT_TOKENS` | Limit MCP output size |

## Common patterns

### One-off analysis
```bash
claude -p "explain this codebase structure"
```

### Continue previous work
```bash
claude -c "now add tests for that feature"
```

### Scripted pipeline
```bash
git diff HEAD~1 | claude -p "write release notes" > CHANGELOG.md
```

### Custom model for session
```bash
claude --model opus "help me architect this system"
```

### Restricted tools
```bash
claude --tools "Read,Grep,Glob" -p "analyze without modifying"
```

### Debug mode
```bash
claude --debug "api,mcp" "test connection"
```

### JSON output for scripting
```bash
claude -p --output-format json "list all TODO comments"
```

### Fork session for experiments
```bash
claude --resume abc123 --fork-session "try a different approach"
```

## Best practices

1. **Use print mode for scripts**: `-p` flag exits after response

2. **Pipe for context**: Feed files, logs, diffs directly via stdin

3. **Name sessions**: Use `-r` with descriptive names for resumable work

4. **Append over replace**: Use `--append-system-prompt` to keep defaults

5. **Restrict tools when needed**: `--tools` and `--disallowedTools` for safety

6. **Set environment variables**: Configure defaults in shell profile

## Common mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Missing `-p` in scripts | Enters interactive mode | Add `-p` flag for print mode |
| Options after server name | MCP CLI parsing fails | Put options before server name |
| Wrong session resume | Can't find session | Use exact ID or unique name prefix |
| Model name typo | Falls back to default | Use aliases: `sonnet`, `opus`, `haiku` |

## Related

- [Settings](settings.md) - Configuration files
- [Hooks](hooks.md) - Automation on events
- [MCP](mcp.md) - Model Context Protocol servers
- [Agents](agents.md) - Subagent configuration
