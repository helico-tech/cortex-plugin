---
name: feature-lite
description: Small feature — implement and review without design or planning ceremony
---

# Feature Lite Workflow

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
    - `hands-off` — Only stop for blockers
    - `during-unknowns` — Ask me when something is unclear or ambiguous
    - `every-step` — Check in with me at every step
  - inject-into: [implement]

## Steps

### step:implement
- command: implement
- on-complete: step:review

### step:review
- command: review
- check: verdict
- on-pass: done
- on-fail: step:implement
