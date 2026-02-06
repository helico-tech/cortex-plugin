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

1. Check if `.cortex/workflows/{workflow-name}.md` exists (project workflow)
2. Check if the name matches a built-in workflow below
3. If found in BOTH, **stop with an error**: "Workflow '{workflow-name}' exists in both the plugin and the project. Project workflows must use unique names to avoid shadowing."
4. If found in neither, list available workflows and ask the user to pick one.

**If it's a project workflow**, read the definition from `.cortex/workflows/{workflow-name}.md`.

**If it's a built-in workflow**, use the definition directly from below — no file search needed.

---

### Built-in: feature

Full feature development — design through validation with review feedback loop.

**Params:**
- **execution-mode**: choice — `auto-chain` | `guided` | `reference`
- **user-involvement**: choice — `hands-off` | `during-unknowns` (default) | `every-step` — inject-into: [design, implement]

**Steps:**
1. `step:design` → command: design → on-complete: step:plan
2. `step:plan` → command: plan → on-complete: step:implement
3. `step:implement` → command: implement (task-selection: all-remaining) → on-complete: step:review
4. `step:review` → command: review → check: verdict → on-pass: step:validate → on-fail: step:implement
5. `step:validate` → command: validate → check: verdict → on-pass: done → on-fail: step:implement

---

### Built-in: feature-lite

Small feature — implement and review without design or planning ceremony.

**Params:**
- **execution-mode**: choice — `auto-chain` | `guided` | `reference`
- **user-involvement**: choice — `hands-off` | `during-unknowns` (default) | `every-step` — inject-into: [implement]

**Steps:**
1. `step:implement` → command: implement → on-complete: step:review
2. `step:review` → command: review → check: verdict → on-pass: done → on-fail: step:implement

---

### Built-in: hotfix

Fast bug fix — triage and fix, then review. Skips design and planning.

**Params:**
- **execution-mode**: choice — `auto-chain` | `guided` | `reference`

**Steps:**
1. `step:fix` → command: fix → check: triage-result → on-complete: step:review → on-needs-design: done (tell user to use `/cortex-team:workflow feature` instead)
2. `step:review` → command: review → check: verdict → on-pass: done → on-fail: step:fix

---

### Built-in: refactor

Investigate, design the refactor, then plan → implement → review → validate.

**Params:**
- **execution-mode**: choice — `auto-chain` | `guided` | `reference`
- **user-involvement**: choice — `hands-off` | `during-unknowns` (default) | `every-step` — inject-into: [refactor, implement]

**Steps:**
1. `step:refactor` → command: refactor → on-complete: step:plan
2. `step:plan` → command: plan → on-complete: step:implement
3. `step:implement` → command: implement (task-selection: all-remaining) → on-complete: step:review
4. `step:review` → command: review → check: verdict → on-pass: step:validate → on-fail: step:implement
5. `step:validate` → command: validate → check: verdict → on-pass: done → on-fail: step:implement

---

### Built-in: maintenance

Codebase gardening — audit, check standards, tidy up, capture learnings.

**Params:**
- **execution-mode**: choice — `auto-chain` | `guided` | `reference`
- **audit-scope**: string — "What area should be maintained?" — inject-into: [audit, tidy]

**Steps:**
1. `step:audit` → command: audit (audit-focus: all) → on-complete: step:conform
2. `step:conform` → command: conform (conform-focus: both) → on-complete: step:tidy
3. `step:tidy` → command: tidy (tidy-focus: all) → on-complete: step:curate
4. `step:curate` → command: curate → on-complete: done

---

### Built-in: spike

Exploratory spike — investigate then design. Stops before implementation.

**Params:**
- **execution-mode**: choice — `auto-chain` | `guided` | `reference`
- **user-involvement**: choice — `hands-off` | `during-unknowns` (default) | `every-step` — inject-into: [design]

**Steps:**
1. `step:investigate` → command: investigate (investigation-focus: both) → on-complete: step:design
2. `step:design` → command: design → on-complete: done

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

**Context management — this is critical for multi-step workflows:**

Each workflow step spawns a cortex-runner command which spawns multiple agents. Without discipline, context explodes.

1. **Each step runs as a Task tool subagent.** The subagent handles the entire cortex-runner flow (all 7 steps: params, memory, artifacts, agents, artifact production, journal, context proposals) and returns only a short summary.
2. **Instruct the subagent to complete all steps but keep its return value short.** Add to every step's Task prompt: "Complete ALL 7 cortex-runner steps including journal writing and context proposals. Return only: step name, artifact filename produced, journal entry filename, verdict (if applicable), and a 2-3 line summary. Do NOT return full agent outputs."
3. **Between steps, your only state is the workflow-state artifact.** Read it fresh at the start of each step. Do not rely on conversation memory of previous steps.
4. **The artifacts ARE the handoff.** Step N produces an artifact file. Step N+1 consumes that artifact file. You don't need to relay content between them — cortex-runner handles artifact loading.

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
