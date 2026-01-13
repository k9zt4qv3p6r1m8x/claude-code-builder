---
description: Update extension-toolkit documentation from code.claude.com/docs/llms.txt
allowed-tools: WebFetch, Read, Write, Edit, Glob, Grep
---

# Update Extension Toolkit Documentation

Update the `extension-toolkit` skill reference files from official documentation.

## Instructions

1. **Fetch llms.txt** to get the documentation index:
   - URL: https://code.claude.com/docs/llms.txt
   - Extract all available pages

2. **Reference file mapping**:
   | Reference File | Source Pages |
   |----------------|--------------|
   | skills.md | /en/skills.md |
   | agents.md | /en/sub-agents.md |
   | plugins.md | /en/plugins.md, /en/plugins-reference.md |
   | hooks.md | /en/hooks.md, /en/hooks-guide.md |
   | commands.md | /en/slash-commands.md |
   | mcp.md | /en/mcp.md |
   | claude.md | /en/memory.md |
   | settings.md | /en/settings.md |
   | cli.md | /en/cli.md |

3. **For each page**:
   - Fetch content from `https://code.claude.com/docs{path}`
   - Update corresponding reference file in plugin's `skills/extension-toolkit/references/`

4. **Update actions**:
   - Update reference files with new content
   - Maintain existing structure and style
   - Add new sections where appropriate
   - Update SKILL.md if there are new components

5. **Final output**:
   - List of pages checked
   - Files updated

## Notes

- Always preserve existing markdown format
- Flag if there are new component types to add
- Check if llms.txt has new pages not yet covered
