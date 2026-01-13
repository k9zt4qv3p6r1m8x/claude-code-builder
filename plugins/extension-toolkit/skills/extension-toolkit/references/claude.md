# CLAUDE.md Reference

> Project instructions and context loaded automatically at startup.

## Overview

CLAUDE.md is a special file that Claude automatically pulls into context when starting a conversation. It provides persistent instructions, documentation, and context that Claude remembers across sessions.

**Use CLAUDE.md for:**
- Common bash commands and workflows
- Code style guidelines
- Testing instructions
- Repository conventions (branch naming, merge vs rebase)
- Developer environment setup
- Project-specific warnings or behaviors
- Any information you want Claude to remember

## File Location

| Scope | Path | Applies to |
|-------|------|------------|
| Project root | `CLAUDE.md` | Anyone in this repository |
| Project .claude | `.claude/CLAUDE.md` | Anyone in this repository |
| Project local | `CLAUDE.local.md` | You only (gitignored) |
| User | `~/.claude/CLAUDE.md` | You, across all projects |
| Parent dirs | `../CLAUDE.md` | Auto-pulled in subdirectories |
| Child dirs | `subdir/CLAUDE.md` | On-demand when working there |
| Enterprise | System paths | All users in organization |

Multiple CLAUDE.md files are merged. More specific files add to or override broader ones.

## File Structure

Plain markdown with no required format:

```markdown
# Project Name

## Quick Commands

```bash
npm run dev      # Start development server
npm run test     # Run tests
npm run build    # Production build
```

## Code Style

- Use TypeScript strict mode
- Prefer functional components
- Use 2-space indentation

## Architecture

- `/src/components` - React components
- `/src/utils` - Utility functions
- `/src/api` - API clients
```

No frontmatter required. Write naturally as documentation.

## Recommended Sections

| Section | Purpose |
|---------|---------|
| Quick Commands | Common commands with descriptions |
| Code Style | Language conventions, formatting |
| Workflow | How to build, test, deploy |
| Architecture | Key files, patterns, structure |
| Environment | Setup instructions, dependencies |
| Gotchas | Warnings, unexpected behaviors |

## Writing Guidelines

1. **Be concise** - Claude reads this every session. Shorter is better.
2. **Be specific** - "Use ES modules" is better than "Follow modern practices"
3. **Use imperative mood** - "Run `npm test` before committing" not "You should run..."
4. **Include actual commands** - Show exact commands, not descriptions
5. **Organize with headers** - Makes scanning easier
6. **Update regularly** - Add learnings using `#` key during sessions

## The # Command

Press `#` during a Claude Code session to give Claude an instruction it will automatically incorporate into the relevant CLAUDE.md.

Use this to:
- Document commands you just ran
- Add style rules after corrections
- Note project-specific patterns
- Record environment setup steps

## Local vs Shared

| File | Committed | Use for |
|------|-----------|---------|
| `CLAUDE.md` | Yes | Team-shared instructions |
| `.claude/CLAUDE.md` | Yes | Team-shared, hidden from root |
| `CLAUDE.local.md` | No | Personal preferences, local paths |

Use `.local.md` for:
- Machine-specific paths
- Personal workflow preferences
- Temporary instructions
- Sensitive information

## Imports

CLAUDE.md files can import other files using `@` syntax:

```markdown
See @README for project overview.
See @package.json for available commands.

# Additional Instructions
- Git workflow: @docs/git-instructions.md
```

Import from home dir for personal instructions:
```markdown
# Individual Preferences
- @~/.claude/my-project-instructions.md
```

**Import rules:**
- Max depth of 5 hops for recursive imports
- Imports not evaluated inside code spans or blocks
- Use relative paths from the CLAUDE.md file location

## Monorepo Support

Place CLAUDE.md at multiple levels:
```
root/
├── CLAUDE.md           # Repo-wide instructions
├── frontend/
│   └── CLAUDE.md       # Frontend-specific
├── backend/
│   └── CLAUDE.md       # Backend-specific
└── shared/
    └── CLAUDE.md       # Shared utilities
```

Both root and subdirectory CLAUDE.md files load when in that subdirectory.

## Modular Rules

For larger projects, use `.claude/rules/` directory:

```
.claude/rules/
├── code-style.md       # Code style guidelines
├── testing.md          # Testing conventions
└── security.md         # Security requirements
```

Rules are loaded based on file paths being worked on.

### Path-Specific Rules

Scope rules to specific files with frontmatter:

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules

- All endpoints must include input validation
- Use standard error response format
```

Brace expansion is supported: `src/**/*.{ts,tsx}`

### User-Level Rules

Personal rules can be placed in `~/.claude/rules/` for rules that apply across all your projects.

Symlinks are supported in `.claude/rules/` directory.

## Initialization

Run `/init` to generate CLAUDE.md automatically:
```
/init
```

Claude analyzes the codebase and creates initial documentation.

## Example CLAUDE.md

```markdown
# My Project

## Quick Commands

```bash
npm run dev          # Start dev server on :3000
npm run test         # Run Jest tests
npm run lint         # ESLint + Prettier
npm run build        # Production build
```

## Code Style

- TypeScript strict mode
- React functional components with hooks
- Prefer named exports
- 2-space indentation
- Single quotes for strings

## Testing

- Run `npm test` before committing
- Each component needs a `.test.tsx` file
- Use React Testing Library, not Enzyme

## Git Workflow

- Branch from `main`
- Use conventional commits (feat:, fix:, chore:)
- Squash merge to main
- Delete branches after merge

## Architecture

```
src/
├── components/     # Reusable UI components
├── pages/          # Route components
├── hooks/          # Custom React hooks
├── utils/          # Pure utility functions
├── api/            # API client and types
└── styles/         # Global styles
```

## Gotchas

- Don't import from `@/styles` in components (causes HMR issues)
- API responses are camelCase, DB is snake_case
- Tests must run in isolation (no shared state)
```

## Best Practices

1. **Start small** - Begin with essentials, expand as needed
2. **Keep it scannable** - Use headers, short paragraphs, bullets
3. **Include examples** - Show correct patterns, not just rules
4. **Document the why** - Brief explanations help Claude apply rules
5. **Version control it** - Commit changes with related code
6. **Review periodically** - Remove outdated instructions

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Too long | Wastes context | Keep under 500 lines, split if needed |
| Too vague | Not followed consistently | Be specific with commands and patterns |
| Never updated | Stale info misleads Claude | Review and update regularly |
| Duplicated info | Same content in multiple files | Use hierarchy appropriately |
| Sensitive data | Secrets in committed file | Use CLAUDE.local.md |

## Related

- [Settings](settings.md) - Permissions and configuration
- [Skills](skills.md) - Detailed knowledge for specific tasks
- [Commands](commands.md) - Reusable prompts with `/name`
