---
description: Run a workflow — orchestrate multiple commands through a defined flow with state tracking across sessions
argument-hint: Workflow name (e.g. 'feature')
---

# Workflow

This command orchestrates multi-step workflows. It does NOT use the cortex-runner skill — it is its own orchestrator.

## Params

- **workflow-name**:
  - type: string
  - question: "Which workflow? (e.g. 'feature')"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.

## Finding the Workflow Definition

1. Search `.cortex/workflows/{workflow-name}.md` in the project (project workflows)
2. Search this plugin's `workflows/{workflow-name}.md` (plugin workflows)
3. If found in BOTH locations, **stop with an error**: "Workflow '{workflow-name}' exists in both the plugin and the project. Project workflows must use unique names to avoid shadowing."
4. If found in neither, list available workflows from both locations and ask the user to pick one.

**Available plugin workflows:** `feature`, `hotfix`, `refactor`, `maintenance`, `spike`

## Starting or Resuming

Search `.cortex/artifacts/` for any `workflow-state` artifact where `workflow: {workflow-name}` and `status: active`.

**If found (resume):**
- Show the current state: which steps are done, which is current, which are pending
- Ask: "Resume this workflow for {feature-id}, or start a new run?"
- If resuming, pick up from `current-step`

**If not found (new run):**
- Collect workflow-level params (execution-mode, plus any defined in the workflow's Params section)
- The feature-id will be created by the first step (design command assigns it)
- Create the workflow-state artifact after the first step completes

## Execution Modes

### auto-chain

Run each step as a subagent using the Task tool:

1. Read the current step's command definition
2. Spawn a subagent that follows the cortex-runner skill flow for that command
3. Pass the feature-id and any default params from the workflow step
4. Wait for completion
5. Read the produced artifact's frontmatter to check transition conditions
6. Update the workflow-state artifact
7. Follow the transition to the next step
8. Repeat until `done` or a blocker

If a step errors (not verdict-fail, but actual errors), pause and ask the user what to do.

**Context management:** After each step completes, summarize the result in 2-3 lines and discard the step's detail from your working context. The artifact file has the full record — you don't need to carry it. Read the workflow-state artifact at the start of each step to know where you are, not your memory of previous steps.

### guided

For each step:

1. Present what's about to happen: which command, what it will do, what artifacts it consumes/produces
2. Show context: what was produced in prior steps, any relevant state
3. Wait for user confirmation: "Run this step?" / "Skip" / "Abort workflow"
4. On confirmation, execute the step (same as auto-chain but one at a time)
5. After completion, show the result and present the next step

### reference

Show the complete workflow with current position:

1. Display all steps with their status (done/current/pending)
2. Highlight the current step
3. Show the exact command to run: `/cortex-team:{command} {feature-id}`
4. After the user runs the command manually and returns, update the state by checking if the expected artifact was produced

## Workflow State Artifact

After each step, write/update `.cortex/artifacts/{feature-id}.workflow-state.md`:

```markdown
---
artifact: workflow-state
feature: {{feature-slug}}
feature-id: {{feature-id}}
status: active
command: workflow
workflow: {{workflow-name}}
created: YYYY-MM-DD
current-step: {{current-step}}
execution-mode: {{execution-mode}}
---

# Workflow State: {{workflow-name}} / {{feature-id}}

## Progress

| Step | Command | Status | Artifact Produced | Result |
|---|---|---|---|---|
| step:design | design | done/running/pending | filename or — | complete/pass/fail/— |
| step:plan | plan | ... | ... | ... |
| ... | ... | ... | ... | ... |

## History

- YYYY-MM-DD HH:MM — step:design completed
- YYYY-MM-DD HH:MM — step:plan completed
- ...

## Next

(description of what happens next based on current state and transitions)
```

When the workflow reaches `done`, set `status: completed` in the frontmatter.

## Transition Logic

For each completed step, determine the next step:

- **`on-complete: step:X`** — the step produced its artifact successfully. Go to step X.
- **`on-pass: step:X`** — read the produced artifact's frontmatter. If `verdict: pass`, go to step X.
- **`on-fail: step:Y`** — read the produced artifact's frontmatter. If `verdict: fail`, go to step Y.
- **`done`** — workflow is complete. Mark workflow-state as completed. Congratulate the user.

If a step has `check: verdict` but the artifact has no verdict field, treat it as an error and ask the user.
