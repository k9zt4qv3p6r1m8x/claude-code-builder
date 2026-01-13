# Claude Code Builder

Plugins for building and extending Claude Code.

## Installation

Add the marketplace to Claude Code:

```bash
/plugin marketplace add k9zt4qv3p6r1m8x/claude-code-builder
```

Then browse and install plugins via `/plugin` menu.

## Plugins

### Development

| Plugin | Description |
|--------|-------------|
| [extension-toolkit](./plugins/extension-toolkit) | Build Claude Code extensions: skills, agents, commands, hooks, and plugins |

---

## extension-toolkit

Expert toolkit for creating Claude Code extensions.

**Install:**
```bash
/plugin install extension-toolkit
```

**What it does:**
- Guides you through creating skills, agents, commands, hooks, and plugins
- Includes reference documentation for all Claude Code components
- Validates your extensions for errors
- Auto-updates docs from official sources

**Commands:**
| Command | Description |
|---------|-------------|
| `/extension-toolkit-validate` | Validate components in your project |
| `/extension-toolkit-update` | Update reference docs |

**Usage:**

Once installed, use the agent to create extensions. Example:

```
You: @plugins/extension-toolkit/ I need a skill for Next.js data security best practices.
     It should cover Data Access Layer patterns, Server Actions security,
     and input validation. Use this as reference:
     https://nextjs.org/docs/app/guides/data-security

Claude: [Fetches documentation]
        [Creates .claude/skills/nextjs-security/SKILL.md]
        [Creates .claude/skills/nextjs-security/references/data-access-layer.md]
        [Creates .claude/skills/nextjs-security/references/server-actions.md]
        [Runs /extension-toolkit-validate]
        Done! Created nextjs-security skill with 2 reference files.
```

---

Built with [Zed](https://zed.dev)
