---
name: feature
description: Full feature development — design through validation with review feedback loop
---

# Feature Workflow

## Params

- **execution-mode**:
  - type: choice
  - question: "How should this workflow run?"
  - options:
    - `auto-chain` — Run all steps automatically, only stop for required params or blockers
    - `guided` — Present each step, let me confirm before running
    - `reference` — Just show me what's next, I'll invoke commands myself

- **user-involvement**:
  - type: choice
  - question: "How involved do you want to be during each step?"
  - options:
    - `hands-off` — Work autonomously, only interrupt for blockers
    - `during-unknowns` — Check in when you hit ambiguity or need a decision
    - `every-step` — Walk me through everything
  - default: during-unknowns
  - inject-into: [design, implement]

## Steps

### step:design
- command: design
- on-complete: step:plan

### step:plan
- command: plan
- on-complete: step:implement

### step:implement
- command: implement
- params:
  - task-selection: all-remaining
- on-complete: step:review

### step:review
- command: review
- check: verdict
- on-pass: step:validate
- on-fail: step:implement

### step:validate
- command: validate
- check: verdict
- on-pass: done
- on-fail: step:implement
