---
description: Create a task list from a design document — break it into parallelizable tasks assigned to team members
argument-hint: Feature ID (e.g. 0001-auth-flow)
---

# Plan

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: tasks
- consumes: design

## Params

- **feature-id**:
  - type: string
  - question: "Which feature? Provide the feature ID (e.g. 0001-auth-flow)"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask. If the user gives just a slug, search `.cortex/artifacts/` for a matching design file.

## Agents

1. **cortex-team:architect** — Break the design into concrete implementation tasks with dependencies
2. **cortex-team:implementer** — Review tasks for feasibility, estimate sizes, flag unclear requirements
3. **cortex-team:tester** — Add testing tasks, ensure test strategy from design is covered in the plan

Collaboration style: **sequential refinement**. Architect creates initial breakdown. Implementer reviews for feasibility and adds detail. Tester ensures testing tasks are included. Each refines the previous output.

## Task

Create a task list for feature "{{feature-id}}" based on its design document.

**Project context**: (inject loaded context memory)
**Design document**: (inject consumed design artifact)

### Phase 1: Task Breakdown

Launch the **cortex-team:architect** with the design document:

- Break the design into concrete, actionable tasks
- Identify dependencies between tasks (what blocks what)
- Identify which tasks can run in parallel
- Assign each task to the appropriate agent (implementer, tester, writer, etc.)
- Keep tasks small enough to complete in one session

### Phase 2: Feasibility Review

Launch the **cortex-team:implementer** to review the task list:

- Are the tasks specific enough to implement without guessing?
- Are any tasks too large and should be split?
- Are dependencies correct?
- Estimate size: small (< 30 min), medium (30-60 min), large (> 60 min)
- Flag any unclear requirements that need resolving before implementation

### Phase 3: Test Coverage

Launch the **cortex-team:tester** to review and add test tasks:

- Ensure every test from the design's Test Plan has a corresponding task
- Add test tasks at appropriate points in the dependency graph (not all at the end)
- Verify testing tasks are integrated into the implementation flow, not bolted on

### Phase 4: Produce Task List

Write the artifact to `.cortex/artifacts/{{feature-id}}.tasks.md` with this EXACT structure:

```markdown
---
artifact: tasks
feature: {{feature-slug}}
feature-id: {{feature-id}}
status: active
command: plan
created: YYYY-MM-DD
source: {{feature-id}}.design.md
agents: [architect, implementer, tester]
---

# Tasks: {{feature-name}}

Design: {{feature-id}}.design.md

## Task List

### TASK-0001: (task title)
- **Status**: pending
- **Agent**: (implementer/tester/writer)
- **Size**: small/medium/large
- **Depends on**: (TASK-NNNN or none)
- **Parallel**: (can run alongside TASK-NNNN, or sequential)
- **Covers**: REQ-NNN, DEC-NNN, TEST-NNN (from design)
- **Description**: (specific, actionable description — what to do, where, how)

### TASK-0002: ...

## Dependency Graph

(text-based dependency visualization showing parallelization opportunities)

## Execution Order

(suggested order of execution respecting dependencies, grouped into parallel batches)

Batch 1 (parallel):
- TASK-0001
- TASK-0003

Batch 2 (parallel, after batch 1):
- TASK-0002
- TASK-0004

...

## Unresolved Items
(anything from the design's Open Questions that affects task planning)
```
