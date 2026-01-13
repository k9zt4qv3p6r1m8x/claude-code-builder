---
name: extension-toolkit
description: |
  Expert system for creating and configuring Claude Code extensions. Use when:
  - Creating skills, agents, commands, hooks, or plugins
  - Setting up .claude/ directory structure
  - Configuring MCP servers or settings.json
  - Writing CLAUDE.md project instructions
  - Understanding Claude Code's extension architecture
---

# Extension Toolkit

Expert reference for extending Claude Code with custom components.

## Quick Reference

| Need | Component | Location |
|------|-----------|----------|
| Auto-applied knowledge | [Skill](references/skills.md) | `.claude/skills/<name>/SKILL.md` |
| Explicit `/name` prompt | [Command](references/commands.md) | `.claude/commands/<name>.md` |
| Event automation | [Hook](references/hooks.md) | `.claude/settings.json` |
| External tools | [MCP](references/mcp.md) | `.mcp.json` |
| Isolated AI context | [Agent](references/agents.md) | `.claude/agents/<name>.md` |
| Project instructions | [CLAUDE.md](references/claude.md) | `CLAUDE.md` or `.claude/CLAUDE.md` |
| Distributable package | [Plugin](references/plugins.md) | `.claude/plugins/<name>/` |
| Configuration | [Settings](references/settings.md) | `.claude/settings.json` |

## Workflow

1. **Identify the goal** - What does the user want to achieve?
2. **Choose the right component** - Use the table above
3. **Read the reference** - Load the appropriate reference file
4. **Check existing structure** - Read `.claude/` before creating
5. **Create with validation** - Follow patterns from reference
6. **Test locally** - Use `--plugin-dir` for plugins

## Component Comparison

### Skills vs Commands
| Aspect | Skills | Commands |
|--------|--------|----------|
| Trigger | Automatic (Claude decides) | Explicit (`/name`) |
| Structure | Directory with SKILL.md | Single .md file |
| Use for | Complex knowledge, workflows | Quick actions, prompts |

### Skills vs Agents
| Aspect | Skills | Agents |
|--------|--------|--------|
| Context | Adds to current conversation | Separate context window |
| Purpose | Knowledge and workflows | Isolated task execution |
| Invocation | Automatic | Delegation or explicit |

## Critical Rules

1. **Frontmatter must start on line 1** - No blank lines before `---`
2. **Use spaces, not tabs** - YAML requires spaces for indentation
3. **Match names to paths** - `my-skill/` directory → `name: my-skill`
4. **Keep SKILL.md under 500 lines** - Use progressive disclosure
5. **Plugin components at root** - NOT inside `.claude-plugin/`

## File Locations Summary

### Project Scope (commit to git)
```
.claude/
├── skills/<name>/SKILL.md
├── agents/<name>.md
├── commands/<name>.md
├── settings.json
├── rules/<topic>.md
└── CLAUDE.md
.mcp.json
CLAUDE.md
```

### User Scope (personal, all projects)
```
~/.claude/
├── skills/<name>/SKILL.md
├── agents/<name>.md
├── commands/<name>.md
├── settings.json
├── rules/<topic>.md
└── CLAUDE.md
~/.claude.json (MCP)
```

## Creating Components

Before creating any component:
1. Read the appropriate reference file from `references/`
2. Check if similar components already exist
3. Follow the exact structure from reference
4. Validate YAML syntax
5. Test functionality

For detailed specifications, see the reference files linked above.
