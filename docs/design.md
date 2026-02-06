# cortex-team: Design Document

## Problem

Teams using Claude Code lack standardization. Each developer invents their own prompts, workflows, and conventions. Knowledge doesn't accumulate — every session starts from zero. There's no structured way to capture decisions, learn from past work, or ensure consistency across team members.

## Solution

`cortex-team` is a Claude Code plugin that standardizes how a team uses Claude. It provides:

- **Role-based agents** with fixed personas and tool sets
- **Parameterized commands** (prompt templates) that collect structured input before executing
- **Structured memory** that accumulates project knowledge over time
- **A shared execution flow** that every command follows

The plugin owns the standard. Projects own the memory. Knowledge is git-tracked and shared through normal version control.

## Design Decisions

### Commands over prompt-templates directory

Commands in a Claude Code plugin are natively discovered — they show up as `/cortex-team:<name>`. We initially tried a separate `prompt-templates/` directory with a meta-runner command, but commands can't reference sibling files in their own plugin directory (Glob searches from CWD, not the plugin root).

Each template is therefore a command file. The shared execution ceremony lives in a skill.

**Decided:** Each prompt template = a command. Shared logic = a skill.

### Shared execution flow via cortex-runner skill

Every command follows the same 5-step flow. Rather than duplicate these steps across commands, a single `cortex-runner` skill defines the ceremony. Each command says "use the cortex-runner skill" and only defines what's unique: params, agents, and the task.

**Steps:**
1. Collect parameters (defined per command)
2. Load memory (always the same)
3. Execute task with agents (defined per command)
4. Write journal entry (always the same)
5. Propose context updates (always the same, human approves)

### Role-based agents, not task-based

Agents are personas (architect, reviewer, etc.) rather than task-specific (requirements-gatherer, code-implementer). The reason: a role-based agent accumulates identity across workflow steps. The architect agent in step 2 carries context that enriches its work in step 5. Task-based agents start cold every time.

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
Agent-written memory that auto-loads is a bloat machine. Instead, agents propose context updates after each run, and the human approves. This keeps context small, relevant, and trusted.

**Why journal exists at all:**
Raw history has value for reference, git blame, and future curation commands. It just shouldn't be injected into every prompt.

### No config.json

We initially had a `.cortex/config.json` for project name, description, and tech stack. Killed it — that information already lives in `package.json`, `README.md`, and `CLAUDE.md`. Duplicated context goes stale.

### No workflows (yet)

Workflows (multi-step orchestration composing multiple commands) are explicitly deferred. The current primitives (commands, agents, memory) need to be solid first. Workflows compose them later.

### No feature-scoping (yet)

Memory scoped by feature-name was cut. Freeform text as filesystem keys fragments immediately (`auth` vs `authentication` vs `login`). If feature-scoping is needed later, it should use a controlled vocabulary — not freeform input.

## Architecture

### Plugin Structure

```
cortex-team-plugin/
  .claude-plugin/
    plugin.json                ← name, version, description
  agents/
    architect.md               ← role-based agent definitions
    (future: reviewer.md, analyst.md, etc.)
  commands/
    cortex-init.md             ← /cortex-team:cortex-init
    architecture-review.md     ← /cortex-team:architecture-review
    (future: code-review.md, feature-design.md, etc.)
  skills/
    cortex-runner/
      SKILL.md                 ← shared 5-step execution flow
  docs/
    design.md                  ← this file
```

### Project Structure (after cortex-init)

```
any-project/
  .cortex/
    memory/
      context/                 ← curated knowledge (always loaded)
        conventions.md         ← (created by team over time)
        domain-model.md        ← (created by team over time)
        decisions.md           ← (created by team over time)
      journal/                 ← raw history (never auto-loaded)
        2026-02/
          2026-02-06-auth-architecture-review.md
```

### Execution Flow

```
User invokes /cortex-team:architecture-review
        │
        ▼
┌─ Step 1: Collect Params ──────────────────────┐
│  Read command's Params section                 │
│  Ask each question, wait for answer            │
│  Store answers for {{placeholder}} injection   │
└────────────────────────────────────────────────┘
        │
        ▼
┌─ Step 2: Load Memory ─────────────────────────┐
│  Read all .md files from .cortex/memory/context│
│  Summarize what was loaded                     │
└────────────────────────────────────────────────┘
        │
        ▼
┌─ Step 3: Execute ─────────────────────────────┐
│  Hydrate task with params + memory             │
│  Launch agent(s) per command spec              │
│  Respect collaboration style if multi-agent    │
└────────────────────────────────────────────────┘
        │
        ▼
┌─ Step 4: Write Journal ───────────────────────┐
│  Write structured entry to journal/YYYY-MM/    │
│  Automatic, no approval needed                 │
└────────────────────────────────────────────────┘
        │
        ▼
┌─ Step 5: Propose Context Updates ─────────────┐
│  Agent reviews findings                        │
│  Proposes additions to context/ with preview   │
│  Human approves or skips                       │
│  Only writes to context/ on approval           │
└────────────────────────────────────────────────┘
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

## Params
(command-specific parameters with types, questions, options)

## Agents
(which agents to use, collaboration style if multiple)

## Task
(the actual prompt with {{param}} placeholders and memory injection points)
```

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
command: architecture-review
topic: frontend
agents: [architect]
---

## Summary
Reviewed frontend architecture of the dashboard.

## Key Findings
- State management is clean, uses TanStack Query
- Missing error boundaries

## Decisions Made
- Adopt error boundaries at route level

## Open Questions
- Should we add form validation with RHF + Zod?
```

## Future Work

These are explicitly deferred, not forgotten:

- **More agents** — reviewer, business-analyst, implementer, QA
- **More commands** — code-review, feature-design, implementation, retrospective
- **Multi-agent collaboration** — debate, parallel, round-robin styles within a single command
- **Workflows** — composing multiple commands into multi-step sequences
- **Feature-scoping for memory** — controlled vocabulary for feature names, scoped loading
- **Curation command** — `/cortex-team:curate` that reads journal entries and proposes context updates in bulk
- **Size awareness** — warn when context/ exceeds a token threshold
