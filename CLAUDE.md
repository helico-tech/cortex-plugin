# CLAUDE.md

## Project Overview

`cortex-team` is a Claude Code plugin for standardizing how a team uses Claude. It provides role-based agents (8 with deliberate tensions), parameterized commands (10 + init), artifacts that chain between commands, and structured memory.

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
agents/                            ← 8 role-based agent definitions (.md)
commands/                          ← 11 slash commands (.md)
skills/cortex-runner/SKILL.md      ← shared 7-step execution flow
docs/design.md                     ← design document
```

## How Commands Work

Every command delegates to the `cortex-runner` skill for shared ceremony:
1. Collect params (defined per command)
2. Load memory from `.cortex/memory/context/`
3. Load consumed artifacts (stop if missing)
4. Execute task with agent teams (defined per command)
5. Produce artifact (4-digit feature ID, required frontmatter)
6. Write journal to `.cortex/memory/journal/`
7. Propose context updates (human approves)

## Core Flow

```
design → plan → implement → review ←→ implement (feedback loop) → validate
```

Standalone: `fix` (triage + fix or route to design), `refactor` (produces design), `tidy` (find-fix-verify), `audit` (read-only health report), `curate` (journals → context updates)

## Agents

scout, architect, pragmatist, implementer, reviewer, tester, researcher, writer — all `model: inherit`. Every command uses teams, never solo agents.

## Artifacts

Stored at `.cortex/artifacts/{NNNN-feature-slug}.{artifact-type}.md` with required YAML frontmatter. Types: design, tasks, review, validation, tidy-report, audit, curation.

## Git Workflow

- Commit after every significant change
- Do NOT use Co-Authored-By in commit messages
- Write concise commit messages focused on the "why"
