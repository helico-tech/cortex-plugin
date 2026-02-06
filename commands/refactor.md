---
description: Investigate what to refactor and produce a design document — then follows the normal design → plan → implement → review → validate flow
argument-hint: Area to refactor
---

# Refactor

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: design
- consumes: none

## Params

- **refactor-area**:
  - type: string
  - question: "What area of the codebase should be refactored? (module, component, pattern, etc.)"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **refactor-goal**:
  - type: string
  - question: "What's the goal of the refactor? (readability, performance, maintainability, remove duplication, etc.)"
  - required: true
- **user-involvement**:
  - type: choice
  - question: "How involved do you want to be?"
  - options:
    - `hands-off` — Work autonomously, only interrupt for blockers
    - `during-unknowns` — Check in when you hit ambiguity or need a decision
    - `every-step` — Walk me through each finding before moving on

## Agents

1. **scout** — Map the current state of the area to refactor: dependencies, usage, test coverage
2. **architect** — Propose the target state: what the refactored code should look like and why
3. **pragmatist** — Challenge: is this refactor worth the cost? What's the minimum change needed?
4. **tester** — Assess test coverage of the refactor area and define what needs testing to ensure nothing breaks

Collaboration style: **sequential with debate**. Scout maps the current state. Architect proposes the target state. Pragmatist challenges the scope. Tester defines the safety net.

## Task

Design a refactor for "{{refactor-area}}" with goal: {{refactor-goal}}

**Project context**: (inject loaded context memory)

### Phase 1: Current State Analysis

Launch the **scout**:
- Map the area to refactor: all files, dependencies, consumers
- Identify what depends on this code (blast radius)
- Report existing test coverage for this area
- Find related patterns elsewhere in the codebase

### Phase 2: Target State Design

Launch the **architect** with the scout's findings:
- Propose the target state — what should the refactored code look like?
- Define the migration path: how to get from current to target
- Identify risks: what could break during the refactor?
- Ensure the refactor goal ({{refactor-goal}}) is actually achieved

### Phase 3: Scope Challenge

Launch the **pragmatist**:
- Is the full refactor worth it? What's the minimum change that achieves the goal?
- Can the refactor be done incrementally instead of all at once?
- What's the cost of NOT refactoring? Is it actually causing problems today?

### Phase 4: Safety Net

Launch the **tester**:
- What tests exist for this area? Are they sufficient to catch regressions?
- What tests need to be written BEFORE refactoring (the safety net)?
- Define pass/fail criteria: how do we know the refactor didn't break anything?

### Phase 5: Produce Design Document

Write the artifact to `.cortex/artifacts/{{feature-id}}.design.md` using the same EXACT structure as the `design` command's artifact template.

The design document should frame the refactor as a feature with:
- **Requirements**: the refactor goals as checkable requirements
- **Decisions**: the structural changes and why
- **Implementation approach**: the migration path (incremental if possible)
- **Test strategy**: the safety net — tests to write BEFORE refactoring
- **Risks**: what could break and how to detect it

After the design is produced, tell the user:
> "Refactor design complete. Follow the normal flow:
> 1. `/cortex-team:plan {{feature-id}}` to create the task list
> 2. `/cortex-team:implement {{feature-id}}` to execute
> 3. `/cortex-team:review {{feature-id}}` to verify
> 4. `/cortex-team:validate {{feature-id}}` to confirm"

Respect the user's **{{user-involvement}}** preference throughout.
