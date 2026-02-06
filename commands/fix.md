---
description: Assess and fix a bug — triage size first, then fix directly or route to design for larger issues
argument-hint: Bug description
---

# Fix

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: tasks (for small fixes) or routes to design (for large issues)
- consumes: none

## Params

- **bug-description**:
  - type: string
  - question: "Describe the bug. What's happening? What should happen instead?"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **user-involvement**:
  - type: choice
  - question: "How involved do you want to be?"
  - options:
    - `hands-off` — Assess and fix autonomously
    - `during-unknowns` — Check in on triage decision and unknowns
    - `every-step` — Walk me through assessment and fix

## Agents

1. **scout** — Investigate the bug: reproduce, trace the root cause, assess scope
2. **pragmatist** — Triage: is this a small fix or does it need a design?
3. **implementer** — Fix the bug (if small)
4. **tester** — Write a regression test and verify the fix

Collaboration style: **sequential with triage gate**. Scout investigates. Pragmatist triages. If small: implementer fixes, tester verifies. If large: route to design.

## Task

Assess and fix: "{{bug-description}}"

**Project context**: (inject loaded context memory)

### Phase 1: Investigation

Launch the **scout**:
- Find the relevant code
- Trace the root cause
- Identify all files affected
- Assess the blast radius — how much code is touched?

### Phase 2: Triage

Launch the **pragmatist** with the scout's findings:
- Is this a small, contained fix (1-3 files, clear root cause, low risk)?
- Or is this a larger issue that needs a design (architectural problem, unclear scope, high risk)?
- Make a clear recommendation: `fix-now` or `needs-design`

If **{{user-involvement}}** is `during-unknowns` or `every-step`, present the triage decision to the user for confirmation.

### Path A: Fix Now (small bug)

If triage says `fix-now`:

1. Launch the **implementer** with the scout's findings:
   - Fix the root cause
   - Minimal changes — fix the bug, don't refactor the neighborhood

2. Launch the **tester**:
   - Write a regression test that reproduces the bug and verifies the fix
   - Run existing tests to ensure nothing else broke
   - If tests fail, loop back to implementer

3. Create a task list artifact at `.cortex/artifacts/{feature-id}.tasks.md`:
   - Use a new feature-id with slug derived from the bug description
   - Tasks should reflect what was done (investigation, fix, test)
   - Mark all tasks as `done`

### Path B: Needs Design (large issue)

If triage says `needs-design`:

1. Tell the user:
   > "This bug is larger than a quick fix. It affects (scope summary).
   > Recommend running `/cortex-team:design` to properly design the solution."
2. Provide a suggested feature-name and description for the design command
3. Do NOT attempt to fix it — route to the proper flow

Respect the user's **{{user-involvement}}** preference throughout.
