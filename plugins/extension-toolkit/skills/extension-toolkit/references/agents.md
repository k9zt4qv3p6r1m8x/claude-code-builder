# Agents (Subagents)

> Specialized AI assistants with separate context, custom system prompts, and configurable tool access.

## Overview

Agents (subagents) are pre-configured AI assistants that Claude Code can delegate work to. Each agent runs in its own context window with a custom system prompt, specific tool access, and independent permissions.

**Use agents when you need:**
- Isolated context for complex multi-step operations
- Restricted tool access for specific tasks
- Specialized behavior for a domain
- To preserve main conversation context
- Cost control (route to faster, cheaper models like Haiku)

## File Location

| Scope | Path | Applies to |
|-------|------|------------|
| Project | `.claude/agents/<name>.md` | Anyone in this repository |
| User | `~/.claude/agents/<name>.md` | You, across all projects |
| Plugin | `agents/<name>.md` in plugin | Anyone with plugin installed |
| CLI | `--agents` flag | Current session only |

Priority: CLI > Project > User > Plugin

## File Structure

```markdown
---
name: agent-name
description: When to use this agent
[optional fields]
---

[System prompt - instructions for the agent's behavior]
```

The body after the frontmatter becomes the agent's system prompt.

## Required Fields

| Field | Description |
|-------|-------------|
| `name` | Unique identifier. Lowercase, numbers, hyphens only (max 64 chars). Must match filename without `.md` |
| `description` | When Claude should delegate to this agent. Include action keywords users would naturally say |

## Optional Fields

| Field | Description | Default |
|-------|-------------|---------|
| `tools` | Allowlist of tools (e.g., `Read, Grep, Glob, Bash`) | Inherits all |
| `disallowedTools` | Denylist of tools | None |
| `model` | `sonnet`, `opus`, `haiku`, or `inherit` | `sonnet` |
| `skills` | Comma-separated skills to inject (full content loaded at startup) | None |
| `color` | Background color in UI | Automatic |
| `permissionMode` | Permission handling mode | Inherits |
| `hooks` | Lifecycle hooks (`PreToolUse`, `PostToolUse`, `Stop`) | None |

## Tool Configuration

### Allowlist
```yaml
tools: Read, Grep, Glob
```

### Denylist
```yaml
disallowedTools: Write, Edit
```

### Available Built-in Tools
`Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `WebFetch`, `WebSearch`, `Task`, `Skill`, `NotebookEdit`

## Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Standard authorization checks |
| `acceptEdits` | Auto-accept file modifications |
| `dontAsk` | Auto-deny prompts |
| `bypassPermissions` | Skip all checks (use with caution) |
| `plan` | Read-only exploration mode |

## Skills Injection

Skills listed in the `skills` field are **fully injected** into context at startup:
```yaml
skills: code-standards, security-check
```

The full content of each skill file is included in the subagent's system prompt, not just made available for invocation.

**Important:** Built-in agents (Explore, Plan, general-purpose) cannot access custom skills. Only custom subagents with explicit `skills` field can use skills.

## Lifecycle Hooks

```yaml
---
name: my-agent
description: Agent with hooks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./run-linter.sh"
  Stop:
    - type: command
      command: "./cleanup.sh"
---
```

Supported events in agent frontmatter: `PreToolUse`, `PostToolUse`, `Stop`

Note: The `once: true` option is supported in skills and slash commands, but NOT in agents.

## Built-in Agents

| Name | Model | Tools | Purpose |
|------|-------|-------|---------|
| `Explore` | Haiku | Read-only | Fast codebase discovery |
| `Plan` | Inherit | Read-only | Context for plan mode |
| `general-purpose` | Inherit | All tools | Complex multi-step tasks |
| `Bash` | Inherit | Bash | Terminal commands in separate context |
| `statusline-setup` | Sonnet | Read, Edit | Configure status line settings |
| `claude-code-guide` | Haiku | Glob, Grep, Read, WebFetch, WebSearch | Answer questions about Claude Code |

## CLI Definition

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

Fields for CLI agents:
- `description` (required): When to invoke the subagent
- `prompt` (required): System prompt for behavior
- `tools` (optional): Specific tools; inherits all if omitted
- `model` (optional): `sonnet`, `opus`, or `haiku`

## Foreground vs Background

- **Foreground**: Blocks main conversation, permission prompts passed to you
- **Background**: Concurrent execution, auto-denies unpre-approved permissions (`Ctrl+B`)

Use `/bashes` command to list and manage background tasks.

## Resume and Continue

Subagents can be resumed using their agent ID:

```yaml
# Resume a previous subagent conversation
resume: <agent-id-from-previous-invocation>
```

When resumed, the agent continues with its full previous context preserved. The agent ID is returned when a subagent completes.

## Auto-Compaction

Subagents support automatic context compaction when context fills up. This allows long-running subagents to continue working beyond initial context limits.

## Example: Code Reviewer

```markdown
---
name: code-reviewer
description: Expert code review specialist. Reviews code for quality, security, and maintainability. Use proactively after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code clarity and readability
- Proper error handling
- No exposed secrets
- Input validation
- Test coverage

Provide feedback by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider)
```

## Example: Debugger

```markdown
---
name: debugger
description: Debugging specialist for errors, test failures, and unexpected behavior. Use when encountering issues.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate failure location
4. Implement minimal fix
5. Verify solution

Focus on fixing the underlying issue, not symptoms.
```

## Example: DB Query Validator (with Hooks)

```markdown
---
name: db-reader
description: Execute read-only database queries for data analysis.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access.

You cannot modify data. If asked to INSERT, UPDATE, DELETE, explain you only have read access.
```

## Subagent Hooks in settings.json

Configure hooks that run when specific subagents start or stop:

```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [{"type": "command", "command": "./setup-db.sh"}]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "db-agent",
        "hooks": [{"type": "command", "command": "./cleanup-db.sh"}]
      }
    ]
  }
}
```

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Vague description | Won't trigger | Be specific with action keywords |
| Missing `skills` field | No skill access | Explicitly list needed skills |
| Wrong location | Won't load | `.claude/agents/`, not `.claude/skills/` |
| Name mismatch | Loading errors | `foo.md` must have `name: foo` |
| Long system prompt | Wastes context | Use skills for detailed instructions |
| Using `once: true` in agents | Not supported | Use skills or commands for `once` hooks |

## Best Practices

1. **Write trigger-rich descriptions** with action keywords users would naturally say
2. **Apply least-privilege** for tool access
3. **Keep system prompts focused**, use skills for detailed instructions
4. **Match name to filename** exactly
5. **Use semantic prefixes** for project agents (e.g., `myproject-reviewer`)
6. **Use `model: inherit`** for consistency with main conversation
7. **Use background execution** for long-running tasks that don't need interaction

## Related

- [Skills](skills.md) - Add knowledge without separate context
- [Commands](commands.md) - Explicit `/name` invocation
- [Hooks](hooks.md) - Event automation
- [Settings](settings.md) - Permissions and configuration
- [CLI](cli.md) - `--agents` flag for CLI-defined subagents
