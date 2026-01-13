# Commands (Slash Commands)

> Reusable prompt templates triggered explicitly with `/command-name`.

## Overview

Custom slash commands are Markdown files that define frequently used prompts. When you type `/command-name`, Claude executes the prompt template with any arguments you provide.

Use commands for:
- Repeated workflows you invoke manually
- Quick shortcuts for common tasks
- Parameterized prompts with arguments
- Team-shared prompt templates

Unlike skills (which Claude invokes automatically), commands require explicit user invocation.

## File location

| Scope | Path | Shown as | Applies to |
|-------|------|----------|------------|
| Project | `.claude/commands/<name>.md` | `/project:<name>` | Anyone in this repository |
| Project subdirectory | `.claude/commands/<subdir>/<name>.md` | `/project:<name>` (with subdir in description) | Anyone in this repository |
| User | `~/.claude/commands/<name>.md` | `/user:<name>` | You, across all projects |

Project commands take precedence over user commands with the same name.

## File structure

Markdown file with optional YAML frontmatter followed by the prompt template.

The frontmatter section is enclosed in `---` markers at the top of the file.

The body after frontmatter is the prompt that Claude receives when the command is invoked.

## Frontmatter fields

All fields are optional.

| Field | Description | Default |
|-------|-------------|---------|
| `description` | Brief description shown in `/help` | First line of prompt |
| `allowed-tools` | Tools the command can use | Inherits from conversation |
| `argument-hint` | Hint for expected arguments shown in autocomplete | None |
| `model` | Model to use for this command | Inherits from conversation |
| `context` | Set to `fork` to run in separate subagent context | Inline |
| `agent` | Agent type when `context: fork` (`Explore`, `Plan`, `general-purpose`, or custom agent name) | `general-purpose` |
| `hooks` | Hooks scoped to this command's execution | None |
| `disable-model-invocation` | Prevent `Skill` tool from calling this command | `false` |

## Arguments

### All arguments with $ARGUMENTS

Use `$ARGUMENTS` to capture everything passed after the command name as a single string.

When user types `/command-name value1 value2`, `$ARGUMENTS` becomes `value1 value2`.

### Positional arguments with $1, $2, etc.

Use `$1`, `$2`, `$3` etc. to access specific arguments individually.

When user types `/command-name first second third`:
- `$1` becomes `first`
- `$2` becomes `second`
- `$3` becomes `third`

Use positional arguments when you need to place different arguments in different parts of the prompt.

## Bash execution

Execute bash commands in the prompt using `!` prefix with backticks.

Write `!`​`command here`​` in your prompt. The command output is included in the context sent to Claude.

Important: You must include `allowed-tools` with the `Bash` tool for bash execution to work. Specify the allowed bash commands in the format `Bash(command:*)`.

## File references

Include file contents using `@` prefix followed by the file path.

Write `@path/to/file.js` in your prompt. The file content is included in the context.

You can reference multiple files in the same prompt.

## Namespacing with subdirectories

Use subdirectories to organize related commands. The subdirectory name appears in the command description to distinguish commands with the same name.

Structure: `.claude/commands/<subdirectory>/<name>.md`

Commands in different subdirectories can share the same name.

## Forked context

Set `context: fork` in frontmatter to run the command in a separate subagent context.

Use `agent` field to specify which agent type to use: `Explore`, `Plan`, `general-purpose`, or a custom agent name.

Use forked context when:
- Command performs extensive exploration
- You want to preserve main conversation context
- Command runs long operations

## Scoped hooks

Define hooks in frontmatter under the `hooks` key. These hooks run only during this command's execution and are automatically cleaned up when the command finishes.

Supports `PreToolUse`, `PostToolUse`, and `Stop` events.

The `once: true` option runs the hook only once per session.

## Extended thinking

Commands can trigger extended thinking by including these keywords in the prompt:

| Keyword | Thinking budget |
|---------|-----------------|
| `think` | Standard |
| `think hard` | More |
| `think harder` | Even more |
| `ultrathink` | Maximum |

## Built-in Commands

| Command | Description |
|---------|-------------|
| `/add-dir` | Add additional working directories |
| `/agents` | Manage custom AI subagents |
| `/bashes` | List and manage background tasks |
| `/bug` | Report bugs |
| `/clear` | Clear conversation history |
| `/compact` | Compact conversation to save context |
| `/config` | View or modify settings |
| `/context` | Visualize current context usage as colored grid |
| `/cost` | Show token and cost information |
| `/doctor` | Checks the health of Claude Code |
| `/export [filename]` | Export conversation to file or clipboard |
| `/help` | Get help with using Claude Code |
| `/ide` | Connect to an IDE |
| `/init` | Initialize CLAUDE.md with codebase info |
| `/login` | Switch between accounts |
| `/logout` | Sign out of your account |
| `/mcp` | Manage Model Context Protocol servers |
| `/memory` | Edit CLAUDE.md memory files |
| `/model` | Select or change AI model |
| `/output-style [style]` | Set output style |
| `/permissions` | View or modify permissions |
| `/plugin` | Manage Claude Code plugins |
| `/plugins` | Search and install plugins from marketplaces |
| `/pr-comments` | View pull request comments |
| `/quit` | Exit Claude Code |
| `/remote-env` | Configure remote session environment |
| `/rename <name>` | Rename current session |
| `/review` | Request a code review |
| `/sandbox` | Enable sandboxed bash tool |
| `/security-review` | Security review of pending changes |
| `/stats` | Visualize daily usage, session history, streaks |
| `/statusline` | Set up status line UI |
| `/tasks` | View and manage background tasks |
| `/teleport` | Resume remote session from claude.ai |
| `/terminal-setup` | Install Shift+Enter key binding |
| `/todos` | List current TODO items |
| `/usage` | Show plan usage limits and rate limits |
| `/vim` | Toggle Vim mode |

## Skill Tool Integration

The `Skill` tool (which replaced the separate `SlashCommand` tool) can invoke commands programmatically. Commands have a character budget limit (default 15,000 characters).

Set `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable to adjust the limit.

Use `disable-model-invocation: true` to prevent the Skill tool from invoking a command.

## Best practices

1. **Use descriptive names**: Action-oriented names that describe what the command does

2. **Add argument-hint**: Helps users know what arguments to pass

3. **Include allowed-tools**: Be explicit about what tools the command needs

4. **Keep prompts focused**: One command, one purpose

5. **Use semantic prefixes**: For project commands, prefix with project domain

6. **Document with description**: Always add frontmatter description

7. **Commit project commands**: Share with team via version control

## Common mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Missing `allowed-tools` for bash | Bash commands in prompt won't execute | Add `allowed-tools` with required Bash permissions |
| Vague command name | Hard to remember and find | Use specific, action-oriented names |
| No description | Unclear in `/help` output | Add frontmatter description |
| Complex operations without fork | Clutters main context | Use `context: fork` for long operations |
| Hardcoded values | Not reusable | Use `$ARGUMENTS` or positional params |

## Commands vs Skills

| Aspect | Commands | Skills |
|--------|----------|--------|
| Invocation | Explicit `/name` | Automatic by Claude |
| Trigger | User types command | Description matches request |
| Structure | Single .md file | Directory with SKILL.md + resources |
| Use for | Repeatable prompts | Complex knowledge + workflows |

Use commands for quick, explicit actions. Use skills for knowledge Claude should apply automatically.

## Example: Git Status

```markdown
---
description: Show git status with branch info
allowed-tools: Bash(git:*)
---

Show the current git status including:
- Current branch
- Uncommitted changes
- Untracked files

!`git status`
!`git branch -v`
```

## Example: Code Review

```markdown
---
description: Review code changes in a file
argument-hint: <file-path>
allowed-tools: Read, Grep
context: fork
agent: Explore
---

Review the code in @$1 for:
- Code quality issues
- Potential bugs
- Security concerns
- Performance improvements

Provide specific, actionable feedback.
```

## Example: Deploy Preview

```markdown
---
description: Deploy to preview environment
allowed-tools: Bash(npm:*), Bash(git:*)
hooks:
  Stop:
    - type: command
      command: "./scripts/notify-deploy.sh"
---

Deploy the current branch to preview:

1. Run tests first: !`npm test`
2. Build: !`npm run build`
3. Deploy: !`npm run deploy:preview`

Report the preview URL when complete.
```

## Related

- [Skills](skills.md) - Auto-triggered capabilities with instructions
- [Agents](agents.md) - Separate context for delegation
- [Hooks](hooks.md) - Automation on tool events
- [Plugins](plugins.md) - Distribute commands via plugins
