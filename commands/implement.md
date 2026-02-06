---
description: Implement tasks from the task list — execute with the assigned agents, iterate until tasks are complete
argument-hint: Feature ID (e.g. 0001-auth-flow)
---

# Implement

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: tasks (updated)
- consumes: tasks, design

## Params

- **feature-id**:
  - type: string
  - question: "Which feature? Provide the feature ID (e.g. 0001-auth-flow)"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **task-selection**:
  - type: choice
  - question: "Which tasks to implement?"
  - options:
    - `next-batch` — Implement the next batch of unblocked tasks
    - `specific` — Let me choose specific tasks
    - `all-remaining` — Implement all remaining tasks in order

## Agents

1. **scout** — Explore relevant codebase areas before implementation
2. **implementer** — Write the code
3. **tester** — Write tests and verify implementation in-loop (not after)

Collaboration style: **iterative loop per task**. For each task: scout explores → implementer codes → tester verifies → mark done or iterate.

## Task

Implement tasks for feature "{{feature-id}}".

**Project context**: (inject loaded context memory)
**Design document**: (inject consumed design artifact)
**Task list**: (inject consumed tasks artifact)

### Execution Loop

For each task to implement (based on **{{task-selection}}**):

#### 1. Pre-Implementation: Scout

Launch the **scout** to explore the relevant codebase area for this specific task:
- Find the files that need to change
- Understand existing patterns in those files
- Report integration points and conventions to follow

#### 2. Implement

Launch the **implementer** with the scout's findings, the task description, and the relevant design sections:
- Write the code following existing patterns
- Make minimal changes — only what the task requires
- Run existing tests to ensure nothing breaks

#### 3. In-Loop Verification

Launch the **tester**:
- Write the tests specified in the task (if it's a test task) or verify the implementation (if it's a code task)
- Run the tests
- If tests fail: report what failed and loop back to step 2 with the failure details
- If tests pass: confirm the task is complete

#### 4. Update Task Status

After each task is verified:
- Update the task's status in `.cortex/artifacts/{{feature-id}}.tasks.md` from `pending` to `done`
- If the task couldn't be completed, set status to `blocked` and add a note explaining why

### Continue Until Done

Repeat the loop for each task in the selection. After all selected tasks are processed, present a summary:
- Tasks completed
- Tasks blocked (with reasons)
- Tasks remaining
- Any issues discovered during implementation that weren't in the design

### Update the Tasks Artifact

Write the updated task list back to `.cortex/artifacts/{{feature-id}}.tasks.md` with updated statuses. Keep the same frontmatter but update `status` to reflect completion state.
