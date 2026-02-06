---
name: spike
description: Exploratory spike — investigate a topic, then design a solution. Stops before implementation.
---

# Spike Workflow

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
  - question: "How involved do you want to be?"
  - options:
    - `hands-off` — Work autonomously, only interrupt for blockers
    - `during-unknowns` — Check in when you hit ambiguity or need a decision
    - `every-step` — Walk me through everything
  - default: during-unknowns
  - inject-into: [design]

## Steps

### step:investigate
- command: investigate
- params:
  - investigation-focus: both
- on-complete: step:design

### step:design
- command: design
- on-complete: done
