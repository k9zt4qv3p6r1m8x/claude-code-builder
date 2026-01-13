# Hooks Reference

> Shell commands that execute automatically at specific points in Claude Code's lifecycle.

## Overview

Hooks are user-defined shell commands that run at various points during Claude Code's operation. They provide deterministic control over behavior, ensuring certain actions always happen.

**Use hooks for:**
- Automatic formatting after file edits
- Notifications when Claude needs input
- Logging and compliance tracking
- Automated feedback on code conventions
- Blocking modifications to sensitive files
- Custom permission logic

## File Location

Hooks are configured in `settings.json`, not as separate files.

| Scope | Path | Applies to |
|-------|------|------------|
| Project | `.claude/settings.json` | Anyone in this repository |
| Local | `.claude/settings.local.json` | You only, in this repository |
| User | `~/.claude/settings.json` | You, across all projects |

## Configuration Structure

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

### Fields

| Field | Description |
|-------|-------------|
| `matcher` | Tool name pattern (case-sensitive). Use `*` for all tools |
| `type` | `"command"` for bash, `"prompt"` for LLM evaluation, `"agent"` for agentic verification |
| `command` | Shell command to execute (type: command) |
| `prompt` | Prompt for LLM evaluation (type: prompt) |
| `timeout` | Optional timeout in seconds (default: 60) |

### Matcher Patterns

| Pattern | Matches |
|---------|---------|
| `Bash` | Only Bash tool |
| `Edit` | Only Edit tool |
| `Edit\|Write` | Edit or Write tools |
| `*` | All tools |
| Empty string | All events of that type |

## Hook Events

| Event | When it runs | Can block |
|-------|--------------|-----------|
| `PreToolUse` | Before tool calls | Yes (exit 2) |
| `PostToolUse` | After tool calls complete | No |
| `PostToolUseFailure` | After Claude tool execution fails | No |
| `PermissionRequest` | When permission dialog shown | Yes |
| `Notification` | When Claude sends notifications | No |
| `Stop` | When Claude finishes responding | Yes |
| `SubagentStart` | When subagent task starts | No |
| `SubagentStop` | When subagent completes | Yes |
| `PreCompact` | Before context compaction | No |
| `SessionStart` | When session starts/resumes | No |
| `SessionEnd` | When session ends | No |
| `UserPromptSubmit` | When user submits prompt | Yes |

## Hook Types

### Command Type
Execute a shell command:
```json
{
  "type": "command",
  "command": "./scripts/validate.sh",
  "timeout": 30
}
```

### Prompt Type
Use LLM evaluation (for `Stop`, `SubagentStop`, `PreToolUse`, `PermissionRequest`, `UserPromptSubmit`):
```json
{
  "type": "prompt",
  "prompt": "Evaluate if all tasks are complete. Respond with {\"ok\": true} or {\"ok\": false, \"reason\": \"...\"}",
  "timeout": 30
}
```

### Agent Type
Run an agentic verifier with tools for complex verification:
```json
{
  "type": "agent",
  "prompt": "Verify the changes are correct by running tests",
  "tools": ["Read", "Bash"],
  "timeout": 120
}
```

The agent hook type allows complex verification tasks that require tool use (reading files, running commands, etc.).

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success, continue normally |
| 2 | Block the operation (PreToolUse/PermissionRequest only) |
| Other | Error, logged but doesn't block |

## Input (stdin)

Hooks receive JSON data via stdin:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  },
  "tool_use_id": "toolu_01ABC..."
}
```

## Output (stdout/stderr)

- **Exit 0**: stdout shown to user in verbose mode (Ctrl+O)
- **Exit 2**: stderr used as error message, fed back to Claude
- **Other**: stderr shown to user in verbose mode

## JSON Output (Advanced)

Return structured JSON for more control:

### PreToolUse Decision

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "Auto-approved",
    "updatedInput": {
      "command": "modified-command"
    }
  }
}
```

Decisions: `"allow"`, `"deny"`, `"ask"`

### PostToolUse Feedback

```json
{
  "decision": "block",
  "reason": "Explanation for Claude",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "Additional info for Claude"
  }
}
```

### PermissionRequest Control

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "behavior": "allow",
    "updatedInput": {
      "command": "modified-command"
    }
  }
}
```

Or to deny:
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "behavior": "deny",
    "message": "Operation not permitted",
    "interrupt": true
  }
}
```

`interrupt: true` stops Claude from continuing after denial.

### Stop/SubagentStop Control

```json
{
  "decision": "block",
  "reason": "Must continue because..."
}
```

### UserPromptSubmit Context

```json
{
  "decision": "block",
  "reason": "Blocked because...",
  "hookSpecificOutput": {
    "hookEventName": "UserPromptSubmit",
    "additionalContext": "Added context for Claude"
  }
}
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `CLAUDE_PROJECT_DIR` | Project root directory |
| `CLAUDE_ENV_FILE` | File to persist env vars (SessionStart only) |

## Examples

### Auto-format on Edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$FILE\""
          }
        ]
      }
    ]
  }
}
```

### Validate Bash Commands

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.sh"
          }
        ]
      }
    ]
  }
}
```

### Block Sensitive Files

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$TOOL_INPUT\" | grep -q '.env'; then echo 'Cannot edit .env files' >&2; exit 2; fi"
          }
        ]
      }
    ]
  }
}
```

### Notification Alert

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs permission\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

Notification matchers include: `permission_prompt`, `auth_success`, `elicitation_dialog`

### SessionStart Context Loading

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"Project: $(basename $CLAUDE_PROJECT_DIR), Branch: $(git branch --show-current)\""
          }
        ]
      }
    ]
  }
}
```

### Agent-Based Verification

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Verify all tests pass and there are no linting errors",
            "tools": ["Bash", "Read"],
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

## Hooks in Skills/Agents/Commands

Define hooks in frontmatter (only `PreToolUse`, `PostToolUse`, `Stop`):

```yaml
---
name: my-skill
description: Skill with hooks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./validate.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./format.sh"
          once: true
  Stop:
    - type: command
      command: "./cleanup.sh"
---
```

The `once: true` option runs only once per session. Supported in skills and slash commands, NOT in agents.

## Plugin Hooks

In `hooks/hooks.json`:
```json
{
  "description": "Plugin hooks",
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"
          }
        ]
      }
    ]
  }
}
```

## Subagent Hooks

In settings.json for agent lifecycle:

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

## Best Practices

1. **Keep hooks fast** - They run synchronously and block Claude
2. **Handle errors gracefully** - Exit 0 on non-fatal errors
3. **Use specific matchers** - Match only needed tools, not `*`
4. **Log to files, not stdout** - Unless you want Claude to see output
5. **Test manually first** - Run command with sample input
6. **Use project scope for team hooks** - Commit `.claude/settings.json`

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Slow hooks | Blocks Claude | Optimize or run async |
| Wrong exit code | Accidentally blocks | Use 0 for success, 2 to block |
| Overly broad matcher | Runs too often | Use specific tool names |
| Not parsing JSON | Can't access input | Use `jq` to parse stdin |
| Using `once: true` in agents | Not supported | Use skills or commands for `once` hooks |

## Debugging

```bash
claude --debug
```

Shows hook execution details in verbose mode (Ctrl+O).

## Related

- [Settings](settings.md) - Full settings.json configuration
- [Commands](commands.md) - Scoped hooks in commands
- [Skills](skills.md) - Scoped hooks in skills
- [Agents](agents.md) - Agent lifecycle hooks
