---
name: maintenance
description: Codebase gardening — audit health, check standards conformance, tidy up what's easy, capture learnings into context memory
---

# Maintenance Workflow

## Params

- **execution-mode**:
  - type: choice
  - question: "How should this workflow run?"
  - options:
    - `auto-chain` — Run all steps automatically, only stop for required params or blockers
    - `guided` — Present each step, let me confirm before running
    - `reference` — Just show me what's next, I'll invoke commands myself

- **audit-scope**:
  - type: string
  - question: "What area should be maintained? (directory, module, or 'everything')"
  - required: true
  - inject-into: [audit, tidy]

## Steps

### step:audit
- command: audit
- params:
  - audit-focus: all
- on-complete: step:conform

### step:conform
- command: conform
- params:
  - conform-focus: both
- on-complete: step:tidy

### step:tidy
- command: tidy
- params:
  - tidy-focus: all
- on-complete: step:curate

### step:curate
- command: curate
- on-complete: done
