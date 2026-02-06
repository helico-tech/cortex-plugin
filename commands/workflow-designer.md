---
description: Design a project workflow collaboratively — compose commands into a custom flow with transitions and defaults
argument-hint: Workflow name or purpose
---

# Workflow Designer

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: none (writes to `.cortex/workflows/` instead)
- consumes: none

## Params

- **workflow-purpose**:
  - type: string
  - question: "What is this workflow for? Describe the goal or give it a name."
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.

## Agents

1. **architect** — Propose workflow structure based on the goal, suggest command ordering that respects artifact dependencies
2. **pragmatist** — Challenge unnecessary steps, push for the minimal flow that achieves the goal

Collaboration style: **iterative with user**. This is a collaborative design session. The architect proposes, the pragmatist challenges, the user decides. One decision at a time.

## Task

Design a project workflow for: "{{workflow-purpose}}"

**Project context**: (inject loaded context memory)

### Phase 1: Understand the Goal

Ask the user one question at a time to understand:
- What is this workflow trying to achieve?
- What's the starting point? (from scratch, from existing code, from a bug report?)
- What's the end state? (validated feature, quick fix, audit report?)
- Are there any constraints? (must include review, must skip design, etc.)

Do NOT dump all questions at once. One at a time. Build understanding incrementally.

### Phase 2: Present Available Commands

Show the user the full command palette with artifact contracts:

| Command | Consumes | Produces | Purpose |
|---|---|---|---|
| design | — | design | Feature design with requirements, decisions, test strategy |
| plan | design | tasks | Break design into implementable tasks |
| implement | tasks, design | tasks (updated) | Execute tasks with scout→implement→test loop |
| review | tasks, design | review | Adversarial code review with verdict |
| validate | design, tasks, review | validation | Verify requirements met, review resolved |
| fix | — | tasks | Triage and fix a bug (or route to design) |
| refactor | — | design | Investigate and design a refactor |
| tidy | — | tidy-report | Find-fix-verify code quality issues |
| audit | — | audit | Read-only codebase health report |

Highlight which commands make sense for the stated goal.

### Phase 3: Compose Steps

Launch the **architect** to propose an initial workflow structure:
- Order commands to respect artifact dependencies (can't review without tasks)
- Suggest transitions: what happens on pass/fail for verdict-producing commands
- Identify which workflow-level params should exist

Launch the **pragmatist** to challenge:
- Is every step necessary? What's the minimum flow?
- Are there steps that could be optional?
- Is this just the standard `feature` workflow with one tweak? If so, say that.

Present the proposed workflow to the user. Iterate until they're happy:
- Show each step with its transitions
- Let the user add, remove, or reorder steps
- Let the user set default params per step
- Let the user define workflow-level params that inject into steps

### Phase 4: Validate

Before saving, verify:
- **Artifact chain is valid**: every consumed artifact has a producing step upstream
- **No dead ends**: every step has at least one transition (on-complete, on-pass, on-fail, or done)
- **No infinite loops without exit**: every loop has a path that eventually reaches `done`
- **No name collision**: the workflow name doesn't conflict with plugin workflows

Available plugin workflow names: `feature`

If validation fails, show the issue and help the user fix it.

### Phase 5: Save

Write the workflow definition to `.cortex/workflows/{workflow-name}.md` using this format:

```markdown
---
name: {{workflow-name}}
description: {{one-line description}}
---

# {{Workflow Title}}

## Params

(workflow-level params with types, questions, options, defaults, inject-into)

## Steps

### step:{{name}}
- command: {{command-name}}
- params:
  - {{param}}: {{default-value}}
- on-complete: step:{{next}} (for steps without verdicts)
- check: verdict (for steps that produce verdicts)
- on-pass: step:{{next}} (if verdict is pass)
- on-fail: step:{{fallback}} (if verdict is fail)
```

After saving, tell the user:
> "Workflow '{{workflow-name}}' saved to `.cortex/workflows/{{workflow-name}}.md`.
> Run it with: `/cortex-team:workflow {{workflow-name}}`"

### Skip the Cortex-Runner Post-Steps

This command does NOT produce a standard artifact, so skip steps 5-7 of the cortex-runner (produce artifact, journal, propose context). The workflow file IS the output, written directly to `.cortex/workflows/`.
