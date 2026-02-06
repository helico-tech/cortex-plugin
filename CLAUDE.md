# CLAUDE.md

## Project Overview

`cortex-team` is a Claude Code plugin for standardizing how a team uses Claude. It provides role-based agents, parameterized prompt templates, structured memory, and project initialization.

## Development

This is a Claude Code plugin. No build step — just markdown files and a plugin manifest.

Test locally with:
```bash
claude --plugin-dir /Users/avanwieringen/Development/helico/cortex-team-plugin
```

## Plugin Structure

```
.claude-plugin/plugin.json   ← manifest
agents/                       ← role-based agent definitions (.md)
commands/                     ← slash commands (.md)
prompt-templates/             ← parameterized prompt templates (.md)
```

## Git Workflow

- Commit after every significant change
- Do NOT use Co-Authored-By in commit messages
- Write concise commit messages focused on the "why"
