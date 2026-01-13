---
description: Validate Claude Code components (skills, agents, commands, plugins, hooks)
allowed-tools: Read, Glob, Grep, Bash
argument-hint: [path] or leave empty for all
---

# Validate Claude Code Components

Validate structure AND best practices for Claude Code components.

**Scope**: If `$ARGUMENTS` is provided, validate only that path. Otherwise validate all `.claude/` components.

---

## 1. CLAUDE.md Files

**Locations to check**: `./CLAUDE.md`, `./.claude/CLAUDE.md`, `./CLAUDE.local.md`

### Structure
- [ ] File exists (at least one)
- [ ] Valid markdown syntax

### Best Practices
- [ ] **Concise**: Under 200 lines (WARN if over 300)
- [ ] **Human-readable**: No walls of text, uses headers/lists
- [ ] **Actionable**: Contains bash commands, style rules, or workflow tips
- [ ] **No secrets**: No API keys, passwords, tokens

---

## 2. Skills (`.claude/skills/*/SKILL.md`)

### Structure
- [ ] YAML frontmatter starts on line 1 (no blank lines before `---`)
- [ ] Uses spaces, NOT tabs
- [ ] Required: `name`, `description`
- [ ] `name` matches directory name exactly
- [ ] `name` is lowercase-with-hyphens (max 64 chars)

### Best Practices
- [ ] **Under 500 lines**: Use `references/` for detailed docs
- [ ] **Rich description**: Contains action keywords users would say
- [ ] **Progressive disclosure**: Essential info in SKILL.md, details in references/
- [ ] **References exist**: All links to `references/*.md` resolve

---

## 3. Agents (`.claude/agents/*.md`)

### Structure
- [ ] YAML frontmatter starts on line 1
- [ ] Uses spaces, NOT tabs
- [ ] Required: `name`, `description`
- [ ] Filename matches `name` + `.md`
- [ ] `tools` contains valid tool names (if present)
- [ ] `model` is valid: `sonnet`, `opus`, `haiku`, `inherit` (if present)

### Best Practices
- [ ] **Clear description**: Explains WHEN to use, not just WHAT it does
- [ ] **Least privilege**: `tools` allowlist only what's needed
- [ ] **No trigger keywords line**: Keywords should be in description naturally

---

## 4. Commands (`.claude/commands/*.md`)

### Structure
- [ ] Valid YAML frontmatter (if present)
- [ ] Uses spaces, NOT tabs
- [ ] Filename is lowercase-with-hyphens

### Best Practices
- [ ] **Uses $ARGUMENTS**: If command takes input, use `$ARGUMENTS` keyword
- [ ] **Specific instructions**: Clear steps, not vague guidance
- [ ] **Checkable into git**: No personal paths or secrets

---

## 5. Plugins (`.claude-plugin/plugin.json`)

### Structure
- [ ] `plugin.json` is valid JSON
- [ ] `plugin.json` is ONLY inside `.claude-plugin/`
- [ ] Required: `name` field
- [ ] Components at plugin ROOT (not inside `.claude-plugin/`)

### Best Practices
- [ ] **Relative paths**: Use `./` format, not absolute
- [ ] **Use variables**: `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths
- [ ] **Semantic versioning**: `version` follows MAJOR.MINOR.PATCH

---

## 6. Hooks (`.claude/settings.json`)

### Structure
- [ ] Valid JSON
- [ ] Event names are valid: `PreToolUse`, `PostToolUse`, `Notification`, `Stop`, `SubagentStop`, `PreCompact`, `SessionStart`, `SessionEnd`, `UserPromptSubmit`, `PermissionRequest`
- [ ] `type` is `command` or `prompt`

### Best Practices
- [ ] **Scripts exist**: Referenced scripts are present and executable
- [ ] **Matcher patterns valid**: Regex or tool names like `Write|Edit`, `Bash(git commit:*)`
- [ ] **No dangerous patterns**: Avoid `Bash(rm:*)` or `Bash(*:*)`

---

## 7. MCP Servers (`.mcp.json`)

### Structure
- [ ] Valid JSON
- [ ] Each server has `command` or `url`

### Best Practices
- [ ] **No hardcoded secrets**: Use environment variables for API keys
- [ ] **Args are arrays**: `args` should be array, not string

---

## 8. Rules (`.claude/rules/*.md`)

### Structure
- [ ] Valid YAML frontmatter if using `paths` field
- [ ] Glob patterns in `paths` are syntactically valid

### Best Practices
- [ ] **Focused topics**: One rule file per topic (e.g., `typescript-style.md`)
- [ ] **Actionable**: Contains specific do/don't guidance

---

## Valid Tool Names

`Read`, `Write`, `Edit`, `Bash`, `Glob`, `Grep`, `WebFetch`, `WebSearch`, `Task`, `Skill`, `TodoWrite`, `NotebookEdit`

## Valid Hook Events

`PreToolUse`, `PostToolUse`, `PermissionRequest`, `Notification`, `Stop`, `SubagentStop`, `PreCompact`, `SessionStart`, `SessionEnd`, `UserPromptSubmit`
