# CLAUDE.md

## Project Overview

`cortex-team` is a Claude Code plugin for standardizing how a team uses Claude. It provides role-based agents, parameterized commands, and structured memory that accumulates over time.

See `docs/design.md` for the full design document with rationale.

## Development

This is a Claude Code plugin. No build step — just markdown files and a plugin manifest.

Test locally with:
```bash
claude --plugin-dir /Users/avanwieringen/Development/helico/cortex-team-plugin
```

## Plugin Structure

```
.claude-plugin/plugin.json        ← manifest
agents/                            ← role-based agent definitions (.md)
commands/                          ← slash commands / prompt templates (.md)
skills/cortex-runner/SKILL.md      ← shared execution flow (params, memory, storage)
docs/design.md                     ← design document
```

## How Commands Work

Every command delegates to the `cortex-runner` skill for shared ceremony:
1. Collect params (defined per command)
2. Load memory from `.cortex/memory/context/`
3. Execute task with agents (defined per command)
4. Write journal to `.cortex/memory/journal/`
5. Propose context updates (human approves)

## Git Workflow

- Commit after every significant change
- Do NOT use Co-Authored-By in commit messages
- Write concise commit messages focused on the "why"
