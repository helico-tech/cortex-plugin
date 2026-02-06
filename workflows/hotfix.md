---
name: hotfix
description: Fast bug fix — triage and fix, then review. Skips design and planning entirely.
---

# Hotfix Workflow

## Params

- **execution-mode**:
  - type: choice
  - question: "How should this workflow run?"
  - options:
    - `auto-chain` — Run all steps automatically, only stop for required params or blockers
    - `guided` — Present each step, let me confirm before running
    - `reference` — Just show me what's next, I'll invoke commands myself

## Steps

### step:fix
- command: fix
- check: triage-result
- on-complete: step:review
- on-needs-design: done

If the fix command triages the bug as `needs-design`, the workflow ends and tells the user to use `/cortex-team:workflow feature` instead.

### step:review
- command: review
- check: verdict
- on-pass: done
- on-fail: step:fix
