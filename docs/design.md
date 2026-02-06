# cortex-team: Design Document

## Problem

Teams using Claude Code lack standardization. Each developer invents their own prompts, workflows, and conventions. Knowledge doesn't accumulate — every session starts from zero. There's no structured way to capture decisions, learn from past work, or ensure consistency across team members.

## Solution

`cortex-team` is a Claude Code plugin that standardizes how a team uses Claude. It provides:

- **Role-based agents** with deliberate tensions between them
- **Parameterized commands** that use teams of experts (never solo agents)
- **Artifacts** — standardized outputs that chain between commands
- **Structured memory** that accumulates project knowledge over time
- **A shared execution flow** (cortex-runner) that every command follows

The plugin owns the standard. Projects own the memory and artifacts. Everything is git-tracked and shared through normal version control.

## Core Flow

The primary workflow chains commands through artifacts:

```
design → plan → implement → review ←→ implement (feedback loop) → validate
```

Each command consumes artifacts from prior commands and produces its own. This creates a traceable chain from design decisions through to validation.

Additional standalone commands: `fix`, `refactor`, `tidy`, `audit`, `investigate`, `curate` — each with their own artifact contracts.

Workflows orchestrate multiple commands: `/cortex-team:workflow feature` chains design→plan→implement→review→validate with state tracking across sessions.

## Design Decisions

### Commands over prompt-templates directory

Commands in a Claude Code plugin are natively discovered — they show up as `/cortex-team:<name>`. We initially tried a separate `prompt-templates/` directory with a meta-runner command, but commands can't reference sibling files in their own plugin directory (Glob searches from CWD, not the plugin root).

Each template is therefore a command file. The shared execution ceremony lives in a skill.

**Decided:** Each prompt template = a command. Shared logic = a skill.

### Shared execution flow via cortex-runner skill (7 steps)

Every command follows the same 7-step flow. Rather than duplicate these steps across commands, a single `cortex-runner` skill defines the ceremony. Each command says "use the cortex-runner skill" and only defines what's unique: params, agents, and the task.

**Steps:**
1. Collect parameters (defined per command)
2. Load memory (always the same — all `.cortex/memory/context/` files)
3. Load consumed artifacts (check command's `consumes:` declaration, stop if missing)
4. Execute task with agents (defined per command)
5. Produce artifact (auto-increment 4-digit feature ID for new features, write with required frontmatter)
6. Write journal entry (always the same)
7. Propose context updates (always the same, human approves)

### Role-based agents with deliberate tensions

Agents are defined by **role** because an agent in a role can maintain continuity and perspective across workflow steps. The roster is designed with deliberate tensions — agents that productively disagree:

| Tension | Agent A | Agent B | What They Fight About |
|---|---|---|---|
| Build vs Challenge | Architect | Pragmatist | Is this over-engineered? |
| Build vs Break | Implementer | Reviewer | Does this code hold up? |
| Explore vs Judge | Scout | Tester | What's here vs what's missing? |

Every command uses **teams of experts** — multiple agents collaborating, never a single agent working alone. Commands define the collaboration style: sequential, parallel, debate, or iterative loops.

### Artifact system

Artifacts are the backbone of the workflow. They are standardized outputs that chain between commands with explicit contracts.

**Storage:** `.cortex/artifacts/{feature-id}.{artifact-type}.md`

**Feature IDs:** 4-digit sequential prefix + slug: `0001-auth-flow`, `0002-notification-system`. Auto-incremented from the highest existing ID in `.cortex/artifacts/`.

**Artifact types and their producing commands:**

| Artifact | Produced By | Consumed By |
|---|---|---|
| design | design, refactor | plan, implement, review, validate |
| tasks | plan, fix (small) | implement, review, validate |
| review | review | validate, implement (on fail) |
| validation | validate | — |
| tidy-report | tidy | — |
| audit | audit | — |
| investigation | investigate | design (in spike workflow) |
| curation | curate | — |
| workflow-state | workflow | — |

**Frontmatter contract** — every artifact MUST have:
```yaml
---
artifact: {type}
feature: {slug}
feature-id: {NNNN-slug}
status: active
command: {producing-command}
created: YYYY-MM-DD
source: {consumed-artifact-filename or null}
agents: [{participating agents}]
---
```

**Key rules:**
- Commands that `consume` an artifact will **stop execution** if it's missing, directing the user to the producing command
- One file per type per feature, overwritten on re-run (git tracks history)
- The review feedback loop: failed review → findings become new tasks → implement picks them up → review again

### Two-tier memory: context + journal

We evaluated 5 memory categories (conventions, domain-model, decisions, lessons-learned, reflections) and killed them all in favor of two:

| | context/ | journal/ |
|---|---|---|
| **What** | Curated project knowledge | Raw execution history |
| **Loaded** | Always, in full | Never auto-loaded |
| **Written by** | Humans (or agent-proposed, human-approved) | Agents, automatically |
| **Growth** | Bounded by curation | Unbounded (but not loaded) |
| **Purpose** | Inform future runs | Historical reference, curation input |

**Why not 5 categories:**
- `conventions/` and `domain-model/` are just context files — no special treatment needed
- `decisions/` used freeform feature-name as filesystem key — fragmentation guaranteed
- `lessons-learned/` loaded everything always — unbounded bloat
- `reflections/` was per-agent journaling nobody would read

**Why context is human-curated:**
Agent-written memory that auto-loads is a bloat machine. Instead, agents propose context updates after each run (step 7), and the human approves. This keeps context small, relevant, and trusted.

### Workflow system

Workflows compose commands into multi-step flows with branching, state tracking, and configurable execution modes.

**Two levels:**
- **Plugin workflows** — ship with cortex-team in `workflows/`. Standard flows everyone gets.
- **Project workflows** — live in `.cortex/workflows/`. Project-specific compositions created via `/cortex-team:workflow-designer`.
- **Name collisions are errors** — a project workflow cannot shadow a plugin workflow of the same name.

**Execution mode is a runtime param**, not baked into the definition. The same workflow can be:
- `auto-chain` — run all steps automatically as subagents, only stop for params or blockers
- `guided` — present each step, user confirms before running
- `reference` — show what's next, user invokes commands manually

**State is externalized** in a `workflow-state` artifact. This makes workflows survive across sessions — if a session dies or the user comes back tomorrow, the workflow picks up from `current-step`.

**Transitions are artifact-driven:** Steps with `check: verdict` read the produced artifact's frontmatter to determine pass/fail. Steps without verdicts use `on-complete` (succeed if artifact produced). This means the workflow system doesn't need its own judgment — it trusts the artifacts.

**The workflow-designer command** guides users through creating project workflows collaboratively. Architect proposes structure, pragmatist challenges unnecessary steps, user decides. Validates artifact chains (every consumed artifact has a producing step upstream) and prevents infinite loops without exits.

### Project-specific commands

Projects can define their own commands in `.claude/commands/` — Claude Code's native project-level command directory. These commands are auto-discovered and show up as regular slash commands (e.g., `/deploy-staging`).

Project commands can follow cortex conventions by referencing the cortex-runner skill and cortex-team agents, as long as the plugin is installed. This means project commands get the full 7-step flow (params, memory, artifacts, agents, journal, context updates) without reinventing anything.

**Why `.claude/commands/` and not `.cortex/commands/`:** Claude Code already has native project-level command discovery. Building our own would be a worse version of what exists. The cortex-team plugin provides the *framework* (agents, runner, artifacts); the native system provides *discovery*.

**The command-designer command** (`/cortex-team:command-designer`) guides users through creating project commands that follow cortex conventions: proper agent teams, artifact contracts, phased tasks, and cortex-runner integration.

### No config.json

We initially had a `.cortex/config.json` for project name, description, and tech stack. Killed it — that information already lives in `package.json`, `README.md`, and `CLAUDE.md`. Duplicated context goes stale.

### Testing is part of design, not an afterthought

The design command includes a **tester** agent. Test strategy, test plan, pass/fail criteria, and risk areas are all defined at design time. Every test is numbered (TEST-NNN), leveled (unit/integration/e2e), and linked to requirements (REQ-NNN). The plan command then ensures test tasks are woven into the implementation flow, not bolted on at the end.

## Architecture

### Plugin Structure

```
cortex-team-plugin/
  .claude-plugin/
    plugin.json                ← name, version, description
  agents/
    scout.md                   ← maps codebase, reports facts
    architect.md               ← structural decisions, design
    pragmatist.md              ← challenges complexity, YAGNI
    implementer.md             ← writes code, follows patterns
    reviewer.md                ← adversarial code review
    tester.md                  ← test strategy and test writing
    researcher.md              ← web research, docs, APIs
    writer.md                  ← developer documentation
  commands/
    cortex-init.md             ← /cortex-team:cortex-init
    design.md                  ← /cortex-team:design
    plan.md                    ← /cortex-team:plan
    implement.md               ← /cortex-team:implement
    review.md                  ← /cortex-team:review
    validate.md                ← /cortex-team:validate
    fix.md                     ← /cortex-team:fix
    refactor.md                ← /cortex-team:refactor
    tidy.md                    ← /cortex-team:tidy
    audit.md                   ← /cortex-team:audit
    investigate.md             ← /cortex-team:investigate
    curate.md                  ← /cortex-team:curate
    workflow.md                ← /cortex-team:workflow
    workflow-designer.md       ← /cortex-team:workflow-designer
    command-designer.md        ← /cortex-team:command-designer
  workflows/
    feature.md                 ← design→plan→implement→review→validate
    hotfix.md                  ← fix→review (skip design)
    refactor.md                ← refactor→plan→implement→review→validate
    maintenance.md             ← audit→tidy→curate (gardening day)
    spike.md                   ← investigate→design (stop before building)
  skills/
    cortex-runner/
      SKILL.md                 ← shared 7-step execution flow
  docs/
    design.md                  ← this file
```

### Agent Roster

| Agent | Color | Core Trait | Tools | Key Constraint |
|---|---|---|---|---|
| Scout | cyan | Maps codebase, reports facts | Read, Grep, Glob, LS, WebSearch | Does NOT suggest changes |
| Architect | blue | Structural decisions | Read, Grep, Glob, LS | Does NOT write code |
| Pragmatist | yellow | Challenges complexity | Read, Grep, Glob, LS | Tears down, does NOT build |
| Implementer | green | Writes code | Read, Write, Edit, Grep, Glob, LS, Bash, WebSearch | Does NOT question design |
| Reviewer | red | Adversarial code review | Read, Grep, Glob, LS | Does NOT write code |
| Tester | magenta | Test strategy + writing | Read, Write, Edit, Grep, Glob, LS, Bash | Tests only, no production code |
| Researcher | cyan | Web research, docs | WebSearch, WebFetch, Read, Grep, Glob | Reports findings, no opinions |
| Writer | green | Developer docs | Read, Write, Edit, Grep, Glob, LS | Docs only, no code evaluation |

All agents use `model: inherit`.

### Command Contracts

| Command | Consumes | Produces | Agents | Collaboration |
|---|---|---|---|---|
| design | — | design | researcher, scout, architect, pragmatist, tester | Sequential with debate |
| plan | design | tasks | architect, implementer, tester | Sequential refinement |
| implement | tasks, design | tasks (updated) | scout, implementer, tester | Iterative loop per task |
| review | tasks, design | review | reviewer, architect | Parallel then merge |
| validate | design, tasks, review | validation | tester, reviewer | Sequential |
| fix | — | tasks (small) or routes to design | scout, pragmatist, implementer, tester | Sequential with triage gate |
| refactor | — | design | scout, architect, pragmatist, tester | Sequential with debate |
| tidy | — | tidy-report | scout, reviewer, implementer, tester | Find-fix-verify loop |
| audit | — | audit | scout, reviewer, architect, tester | Parallel assessment, merge |
| investigate | — | investigation | scout, researcher | Parallel then merge |
| curate | — | curation | scout, pragmatist, writer | Sequential pipeline |
| workflow | — | workflow-state | — (orchestrator) | Delegates to step commands |
| workflow-designer | — | workflow file (not artifact) | architect, pragmatist | Iterative with user |
| command-designer | — | command file (not artifact) | architect, pragmatist | Iterative with user |

### Project Structure (after cortex-init)

```
any-project/
  .cortex/
    memory/
      context/                 ← curated knowledge (always loaded)
        conventions.md         ← (created by team over time)
        domain-model.md        ← (created by team over time)
      journal/                 ← raw history (never auto-loaded)
        2026-02/
          2026-02-06-0001-auth-flow-design.md
    artifacts/                 ← standardized command outputs
      0001-auth-flow.design.md
      0001-auth-flow.tasks.md
      0001-auth-flow.review.md
      0001-auth-flow.validation.md
      0001-auth-flow.workflow-state.md
    workflows/                 ← project-specific workflow definitions
      hotfix.md                ← (created via workflow-designer)
```

### Command Anatomy

Every command follows this structure:

```markdown
---
description: What this command does
argument-hint: Optional hint for arguments
---

# Command Name

Use the **cortex-runner** skill to execute this template.

## Artifacts
- produces: {artifact-type}
- consumes: {artifact-type(s) or none}

## Params
(command-specific parameters with types, questions, options)

## Agents
(which agents to use, collaboration style)

## Task
(the actual task with {{param}} placeholders and phased execution)
```

### Review Feedback Loop

The review command has a built-in feedback loop:

```
implement → review
              │
        ┌─────┴─────┐
        │            │
      PASS         FAIL
        │            │
        ▼            ▼
    validate    Convert critical findings
                to new tasks in task list
                        │
                        ▼
                   implement (again)
                        │
                        ▼
                    review (again)
```

Failed reviews append new tasks to the existing task list (continuing TASK numbering). The user runs `/cortex-team:implement` again to address them.

### Memory File Formats

**Context files** — freeform markdown, human-maintained:
```markdown
# Conventions

- We use TypeScript strict mode
- API responses follow the envelope pattern
- Tests use vitest with in-memory stores
```

**Journal entries** — structured markdown with frontmatter:
```markdown
---
date: 2026-02-06
command: design
feature-id: 0001-auth-flow
agents: [researcher, scout, architect, pragmatist, tester]
artifact-produced: 0001-auth-flow.design.md
---

## Summary
Designed the authentication flow for the dashboard.

## Key Findings
- Existing session handling is cookie-based
- Need to support both JWT and session tokens

## Decisions Made
- Use JWT for API, sessions for web

## Open Questions
- Token rotation strategy TBD
```

## Future Work

These are explicitly deferred, not forgotten:

- **Feature-scoping for memory** — controlled vocabulary for feature names, scoped loading
- **Size awareness** — warn when context/ exceeds a token threshold
- **Guard-rails** — commands checking if cortex is properly initialized before running
