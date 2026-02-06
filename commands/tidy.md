---
description: Improve code to higher coding standards while ensuring existing functionality is preserved — find issues, fix them, verify nothing breaks
argument-hint: Area to tidy (file, module, or 'recent changes')
---

# Tidy

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: tidy-report
- consumes: none

## Params

- **tidy-scope**:
  - type: string
  - question: "What area should be tidied? (file path, module name, or 'recent changes')"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **tidy-focus**:
  - type: choice
  - question: "What should the focus be?"
  - options:
    - `all` — Everything: naming, structure, error handling, types, tests, standards
    - `readability` — Naming, comments, structure, clarity
    - `robustness` — Error handling, edge cases, validation, types
    - `consistency` — Match conventions used elsewhere in the codebase
    - `standards` — Fix violations of documented coding standards (idioms and architecture)

## Agents

1. **cortex-team:scout** — Map the area: current state, test coverage, conventions in surrounding code
2. **cortex-team:reviewer** — Find code quality issues in the specified area
3. **cortex-team:implementer** — Fix the issues
4. **cortex-team:tester** — Verify nothing broke after each fix

Collaboration style: **find-fix-verify loop**. Scout maps. Reviewer finds issues. Implementer fixes them one by one. Tester verifies after each fix.

## Task

Tidy up "{{tidy-scope}}" with focus on: {{tidy-focus}}

**Project context**: (inject loaded context memory)

### Phase 1: Assessment

Launch the **cortex-team:scout**:
- Map the files in scope
- Identify existing test coverage
- Find conventions in surrounding code (so tidy brings code INTO alignment, not away from it)

Launch the **cortex-team:reviewer** with scout's findings:
- Find code quality issues matching the **{{tidy-focus}}** focus
- If {{tidy-focus}} is `all` or `standards`, and `context/idioms.md` is loaded: include `should`-level and `may`-level idiom violations in the findings (leave `must`-level to the user — those may need design decisions)
- Classify each issue: what it is, where it is, how to fix it
- If the issue violates a documented standard, reference it (IDIOM-NNN)
- Order by impact: biggest improvements first
- Do NOT suggest architectural changes — tidy is cosmetic surgery, not reconstruction

### Phase 2: Fix-Verify Loop

For each issue found, in order of impact:

1. **Implementer**: apply the fix, minimal change
2. **Tester**: run existing tests. If anything fails, revert and skip this fix.
3. Move to next issue

If a fix is risky (touches shared code, changes interfaces), flag it to the user rather than applying it.

### Phase 3: Produce Tidy Report

Write the artifact to `.cortex/artifacts/{feature-id}.tidy-report.md` with this EXACT structure:

```markdown
---
artifact: tidy-report
feature: {{tidy-scope-slug}}
feature-id: {{feature-id}}
status: active
command: tidy
created: YYYY-MM-DD
source: null
agents: [scout, reviewer, implementer, tester]
---

# Tidy Report: {{tidy-scope}}

Focus: {{tidy-focus}}

## Issues Found
(total count)

## Issues Fixed

### FIX-001: (title)
- **Location**: `file:line`
- **Before**: (brief description or code snippet)
- **After**: (brief description or code snippet)
- **Tests**: pass

### FIX-002: ...

## Issues Skipped

### SKIP-001: (title)
- **Location**: `file:line`
- **Reason**: (too risky / would break tests / needs design / out of scope)

## Files Modified
(list of all files changed)

## Test Results
(all tests pass / any failures)
```
