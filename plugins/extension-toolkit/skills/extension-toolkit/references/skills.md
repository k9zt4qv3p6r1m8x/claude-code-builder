# Skills Reference

> Model-invoked capabilities that Claude applies automatically based on context.

## Overview

Skills are markdown files that teach Claude how to do something specific. When your request matches a skill's description, Claude automatically applies it without explicit invocation.

**Use skills for:**
- Team coding standards and workflows
- Complex multi-step procedures
- Domain-specific knowledge
- Reusable patterns with supporting files
- Capabilities requiring scripts or utilities

Unlike commands (which require `/name`), skills are triggered automatically by Claude based on description matching.

## File Location

| Scope | Path | Applies to |
|-------|------|------------|
| Enterprise | Managed settings paths | All users in organization |
| Personal | `~/.claude/skills/<name>/SKILL.md` | You, across all projects |
| Project | `.claude/skills/<name>/SKILL.md` | Anyone in this repository |
| Plugin | `skills/<name>/SKILL.md` in plugin | Anyone with plugin installed |

Priority order: Enterprise > Personal > Project > Plugin (higher row wins when same name)

## File Structure

A skill is a directory containing a `SKILL.md` file with YAML frontmatter and markdown instructions.

```
skill-name/
├── SKILL.md              (required - overview and navigation)
├── reference.md          (detailed docs - loaded when needed)
├── examples.md           (usage patterns - loaded when needed)
└── scripts/
    └── helper.py         (utility script - executed, not loaded)
```

## Required Fields

| Field | Description |
|-------|-------------|
| `name` | Unique identifier. Lowercase, numbers, hyphens only (max 64 chars). Must match directory name |
| `description` | What the skill does and when to use it (max 1024 chars). Claude uses this to decide when to apply the skill |

## Optional Fields

| Field | Description | Default |
|-------|-------------|---------|
| `allowed-tools` | Tools Claude can use without asking when skill is active | Inherits from conversation |
| `model` | Model to use when skill is active (e.g., `claude-sonnet-4-20250514`) | Conversation's model |
| `context` | Set to `fork` to run in separate subagent context | Inline |
| `agent` | Agent type when `context: fork` (`Explore`, `Plan`, `general-purpose`, or custom agent name) | `general-purpose` |
| `hooks` | Hooks scoped to skill lifecycle (`PreToolUse`, `PostToolUse`, `Stop`) | None |
| `user-invocable` | Show in slash command menu | `true` |
| `disable-model-invocation` | Prevent Skill tool from calling | `false` |

## How Skills Work

1. **Discovery**: At startup, Claude loads only the name and description of each skill
2. **Activation**: When request matches description, Claude asks to use the skill (confirmation prompt)
3. **Execution**: Claude follows skill instructions, loading referenced files as needed

## Writing Good Descriptions

The description determines when Claude uses the skill. Answer two questions:

1. **What does this skill do?** List specific capabilities
2. **When should Claude use it?** Include trigger terms users would mention

**Good example:**
```yaml
description: |
  Extract text and tables from PDF files, fill forms, merge documents.
  Use when working with PDF files or when the user mentions PDFs, forms,
  or document extraction.
```

**Bad example:**
```yaml
description: Helps with documents
```

## Progressive Disclosure

Keep `SKILL.md` focused and under 500 lines. For complex skills, use supporting files:

- Put essential instructions in `SKILL.md`
- Put detailed reference material in separate files
- Link to supporting files from `SKILL.md`
- Claude loads additional files only when needed
- Keep references one level deep (avoid deeply nested file chains)

**Reference files in SKILL.md using relative markdown links:**
```markdown
For complete API details, see [reference.md](reference.md)
For usage examples, see [examples.md](examples.md)
```

## Bundled Scripts

Scripts in skill directory can be executed without loading their contents into context. Tell Claude to run the script rather than read it:

```markdown
Run the validation script to check the form:
python scripts/validate_form.py input.pdf
```

Use scripts for:
- Complex validation logic
- Data processing
- Operations requiring consistency across uses

## Tool Restrictions

Use `allowed-tools` to limit which tools Claude can use:

```yaml
---
name: reading-files-safely
description: Read files without making changes
allowed-tools: Read, Grep, Glob
---
```

Or use YAML-style lists:
```yaml
allowed-tools:
  - Read
  - Grep
  - Glob
```

When `allowed-tools` is omitted, the skill doesn't restrict tools. Claude uses standard permission model.

## Forked Context

Set `context: fork` to run skill in isolated subagent context:

```yaml
---
name: code-analysis
description: Analyze code quality and generate detailed reports
context: fork
agent: general-purpose
---
```

Use `agent` field to specify which agent type to use when `context: fork` is set.

## Scoped Hooks

Define hooks under `hooks` key that run during skill lifecycle:

```yaml
---
name: my-skill
description: Skill with lifecycle hooks
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

The `once: true` option runs hook only once per session. After first successful execution, hook is removed.

Hooks defined in a skill are scoped to that skill's execution and automatically cleaned up when skill finishes.

## Visibility Control

| Setting | Slash menu | Skill tool | Auto-discovery | Use case |
|---------|------------|------------|----------------|----------|
| `user-invocable: true` (default) | Visible | Allowed | Yes | Skills users invoke directly |
| `user-invocable: false` | Hidden | Allowed | Yes | Skills Claude uses but users shouldn't invoke manually |
| `disable-model-invocation: true` | Visible | Blocked | Yes | Skills users invoke but not Claude programmatically |

`user-invocable` controls only manual invocation (slash menu). To block programmatic invocation via Skill tool, use `disable-model-invocation: true`.

## Skills and Agents

Subagents do not automatically inherit skills from main conversation. To give a custom subagent access to skills, list them in the agent's `skills` field:

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: Review code for quality and best practices
skills: pr-review, security-check
---
```

The full content of each listed skill is injected into the subagent's context at startup, not just made available for invocation.

**Important:** Built-in agents (Explore, Plan, general-purpose) do not have access to your custom skills. Only custom subagents you define in `.claude/agents/` with an explicit `skills` field can use skills.

## Complete Example

```yaml
---
name: pdf-processing
description: |
  Extract text, fill forms, merge PDFs. Use when working with PDF files,
  forms, or document extraction. Requires pypdf and pdfplumber packages.
allowed-tools: Read, Bash(python:*)
---

# PDF Processing

## Quick start

Extract text:
```python
import pdfplumber
with pdfplumber.open("doc.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

For form filling, see [FORMS.md](FORMS.md).
For detailed API reference, see [REFERENCE.md](REFERENCE.md).

## Requirements

Packages must be installed in your environment:
```bash
pip install pypdf pdfplumber
```
```

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Vague description | Skill doesn't trigger | Include specific capabilities and trigger keywords |
| SKILL.md too long | Wastes context | Split into SKILL.md + reference files (under 500 lines) |
| Deep reference nesting | Partial file loading | Keep references one level deep |
| Wrong file name | Skill won't load | Must be exactly `SKILL.md` (case-sensitive) |
| Name mismatch | Loading errors | Directory name must match `name` field |
| Missing frontmatter | Invalid skill | Must start with `---` on line 1 (no blank lines before) |
| Tabs in YAML | Parse errors | Use spaces for indentation |

## Troubleshooting

**Skill not triggering**: Check description includes keywords users would naturally say. "Helps with documents" is too vague.

**Skill doesn't load**:
- Check file path is correct (exact filename `SKILL.md`, case-sensitive)
- Verify YAML frontmatter syntax (starts with `---` on line 1, spaces not tabs)
- Run `claude --debug` to see skill loading errors

**Plugin skills not appearing**: Clear plugin cache and reinstall:
```bash
rm -rf ~/.claude/plugins/cache
```

## Distribution

| Method | How |
|--------|-----|
| Project | Commit `.claude/skills/` to version control |
| Plugin | Create `skills/` directory in plugin with SKILL.md files |
| Managed | Deploy via managed settings to system directories |

## Related

- [Agents](agents.md) - Separate context for delegation
- [Commands](commands.md) - Explicit `/name` invocation
- [Hooks](hooks.md) - Event automation
- [Plugins](plugins.md) - Distribute skills via plugins
