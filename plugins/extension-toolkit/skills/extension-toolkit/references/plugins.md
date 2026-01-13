# Plugins Reference

> Distributable packages of skills, agents, commands, hooks, MCP servers, and LSP servers.

## Overview

Plugins are self-contained packages that extend Claude Code with additional capabilities. They bundle skills, agents, commands, hooks, MCP servers, and LSP servers into a single distributable unit.

**Use plugins for:**
- Distributing team-specific workflows
- Sharing domain expertise across projects
- Packaging related components together
- Publishing to plugin marketplaces

## Plugin Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json           # Required manifest (ONLY this goes here)
├── commands/                 # Slash commands (at root, NOT in .claude-plugin/)
│   └── my-command.md
├── agents/                   # Agent definitions (at root)
│   └── my-agent.md
├── skills/                   # Skills (at root)
│   └── my-skill/
│       └── SKILL.md
├── hooks/                    # Hook configurations (at root)
│   └── hooks.json
├── .mcp.json                 # MCP servers (optional)
├── .lsp.json                 # LSP servers (optional)
├── scripts/                  # Utility scripts
└── README.md                 # Documentation
```

**CRITICAL:** Components (commands/, agents/, skills/, hooks/) go at plugin root, NOT inside `.claude-plugin/`. Only `plugin.json` goes in `.claude-plugin/`.

## Plugin Manifest

The `.claude-plugin/plugin.json` file:

```json
{
  "name": "my-plugin",
  "description": "What this plugin does",
  "version": "1.0.0",
  "author": {
    "name": "Author Name",
    "email": "author@example.com"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/author/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"]
}
```

### Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Unique identifier (kebab-case) | `"deployment-tools"` |

### Optional Fields

| Field | Description |
|-------|-------------|
| `version` | Semantic version (e.g., `"2.1.0"`) |
| `description` | Brief explanation of purpose |
| `author` | Author object with name, email, url |
| `homepage` | Documentation URL |
| `repository` | Source code URL |
| `license` | License identifier (MIT, Apache-2.0) |
| `keywords` | Discovery tags array |

### Component Path Fields

| Field | Type | Description |
|-------|------|-------------|
| `commands` | string\|array | Additional command files |
| `agents` | string\|array | Additional agent files |
| `skills` | string\|array | Additional skill directories |
| `hooks` | string\|object | Hook config path or inline |
| `mcpServers` | string\|object | MCP config path or inline |
| `lspServers` | string\|object | LSP config path or inline |
| `outputStyles` | string\|array | Additional output style files/directories |

## Plugin Locations

| Source | Path | Discovery |
|--------|------|-----------|
| Local | Any directory | `--plugin-dir` flag |
| Project | `.claude/plugins/` | Automatic |
| User | `~/.claude/plugins/` | Automatic |
| Marketplace | Various | `/plugins` command |

## Installation Scopes

| Scope | Settings File | Use Case |
|-------|---------------|----------|
| `user` | `~/.claude/settings.json` | Personal, all projects |
| `project` | `.claude/settings.json` | Team, via version control |
| `local` | `.claude/settings.local.json` | Project-specific, gitignored |

## Installing Plugins

### From CLI Flag (Development)
```bash
claude --plugin-dir /path/to/my-plugin
claude --plugin-dir ./plugin-a --plugin-dir ./plugin-b
```

### From Marketplace
```bash
/plugins
# Select marketplace, browse, install
```

### CLI Commands
```bash
claude plugin install formatter@marketplace-name
claude plugin install formatter@marketplace-name --scope project
claude plugin uninstall formatter@marketplace-name
claude plugin enable formatter@marketplace-name
claude plugin disable formatter@marketplace-name
claude plugin update formatter@marketplace-name
```

## Plugin Components

### Commands
Plugin commands appear as `/plugin-name:command-name`:
```
commands/
└── deploy.md  →  /my-plugin:deploy
```

### Agents
Plugin agents appear with prefix:
```
agents/
└── reviewer.md  →  my-plugin:reviewer
```

### Skills
Plugin skills appear with prefix:
```
skills/
└── code-review/
    └── SKILL.md  →  my-plugin:code-review
```

### Hooks
In `hooks/hooks.json`:
```json
{
  "description": "Automatic formatting",
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### MCP Servers
In `.mcp.json`:
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

### LSP Servers
In `.lsp.json`:
```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

LSP server fields:
| Field | Description |
|-------|-------------|
| `command` | Command to start the language server |
| `args` | Arguments to pass to the command |
| `transport` | Communication method (default: stdio) |
| `env` | Environment variables |
| `extensionToLanguage` | Map file extensions to language IDs |
| `initializationOptions` | Options passed during initialization |

Available LSP plugins: `pyright-lsp`, `typescript-lsp`, `rust-lsp`

## Environment Variables

| Variable | Description |
|----------|-------------|
| `${CLAUDE_PLUGIN_ROOT}` | Absolute path to plugin directory |
| `${CLAUDE_PROJECT_DIR}` | Project root directory |

## Creating a Plugin

### Step 1: Create Structure
```bash
mkdir -p my-plugin/.claude-plugin
mkdir -p my-plugin/{commands,agents,skills,hooks,scripts}
```

### Step 2: Create Manifest
```json
// my-plugin/.claude-plugin/plugin.json
{
  "name": "my-plugin",
  "description": "Description of capabilities",
  "version": "0.1.0"
}
```

### Step 3: Add Components
Add commands, agents, skills as needed following standard formats.

### Step 4: Test Locally
```bash
claude --plugin-dir ./my-plugin
```

### Step 5: Distribute
- Via marketplace
- Via git repository
- Direct directory sharing

## Marketplace Configuration

### Adding Marketplaces
In `settings.json`:
```json
{
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    }
  }
}
```

### Source Types

| Type | Configuration |
|------|---------------|
| GitHub | `{"source": "github", "repo": "owner/repo"}` |
| URL | `{"source": "url", "url": "https://example.com/marketplace.json"}` |
| NPM | `{"source": "npm", "package": "@scope/package"}` |
| File | `{"source": "file", "path": "/path/to/marketplace.json"}` |

## Plugin Settings

In `settings.json`:
```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "analyzer@security": false
  }
}
```

## Priority Order

Components resolved in order:
1. CLI flag (`--plugin-dir`)
2. Project plugins (`.claude/plugins/`)
3. User plugins (`~/.claude/plugins/`)
4. Marketplace plugins

Higher priority overrides lower when names conflict.

## Plugin Caching

Plugins are cached for performance. File resolution is limited to the plugin directory:
- Path traversal (`../`) is not allowed
- Symlinks pointing outside plugin root are not followed

To work with external dependencies:
- Include dependencies directly in plugin
- Use symlinks that point within plugin directory
- Or use MCP servers that can access external files

Clear plugin cache if needed:
```bash
rm -rf ~/.claude/plugins/cache
```

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Components in `.claude-plugin/` | Won't load | Put at plugin root |
| Missing plugin.json | Plugin won't load | Create `.claude-plugin/plugin.json` |
| Hardcoded paths | Breaks elsewhere | Use `${CLAUDE_PLUGIN_ROOT}` |
| Version not updated | Users don't get updates | Increment on changes |
| Absolute paths | Breaks on other machines | Use relative paths starting with `./` |
| Path traversal in files | Security restriction | Keep files within plugin directory |

## Best Practices

1. **Use semantic versioning** (MAJOR.MINOR.PATCH)
2. **Write clear descriptions** for discoverability
3. **Bundle related components** logically
4. **Include README.md** documenting usage
5. **Use plugin-relative paths** with `${CLAUDE_PLUGIN_ROOT}`
6. **Test before publishing** with `--plugin-dir`
7. **Namespace clearly** with consistent prefixes

## Debugging

```bash
claude --debug
```

Shows:
- Plugin loading details
- Manifest validation errors
- Component registration
- MCP server initialization
- LSP server initialization

## Related

- [Skills](skills.md) - Skill components in plugins
- [Agents](agents.md) - Agent components in plugins
- [Commands](commands.md) - Command components in plugins
- [Hooks](hooks.md) - Hook configuration
- [MCP](mcp.md) - MCP servers in plugins
- [Settings](settings.md) - Plugin settings configuration
