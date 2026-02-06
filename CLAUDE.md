# CLAUDE.md

## Project Overview

`cortex-team` is a Claude Code plugin for standardizing how a team uses Claude. It provides role-based agents (8 with deliberate tensions), parameterized commands (15 + init), workflows that orchestrate commands, artifacts that chain between commands, and structured memory.

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
commands/                          ← 16 slash commands (.md)
workflows/                         ← plugin-level workflow definitions (.md)
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

Exception: `workflow` command is its own orchestrator, doesn't use cortex-runner.

## Core Flow

```
design → plan → implement → review ←→ implement (feedback loop) → validate
```

Standalone: `brainstorm` (open exploration → crystallize → feeds into design), `fix` (triage + fix or route to design), `refactor` (produces design), `tidy` (find-fix-verify), `audit` (read-only health report), `investigate` (fact-finding), `curate` (journals → context updates)

Orchestration: `workflow` (run a multi-step flow), `workflow-designer` (create project workflows), `command-designer` (create project commands)

Project commands live in `.claude/commands/` (native Claude Code discovery). Use `/cortex-team:command-designer` to create them following cortex conventions.

## Workflows

Plugin workflows live in `workflows/`. Project workflows live in `.cortex/workflows/`. Name collisions are errors. Execution mode (auto-chain, guided, reference) is a runtime param. State is tracked in a `workflow-state` artifact.

Standard workflows: `feature` (design→validate), `hotfix` (fix→review), `refactor` (refactor→validate), `maintenance` (audit→tidy→curate), `spike` (investigate→design).

## Agents

scout, architect, pragmatist, implementer, reviewer, tester, researcher, writer — all `model: inherit`. Every command uses teams, never solo agents.

## Artifacts

Stored at `.cortex/artifacts/{NNNN-feature-slug}.{artifact-type}.md` with required YAML frontmatter. Types: brainstorm, design, tasks, review, validation, tidy-report, audit, investigation, curation, workflow-state.

## Git Workflow

- Commit after every significant change
- Do NOT use Co-Authored-By in commit messages
- Write concise commit messages focused on the "why"
