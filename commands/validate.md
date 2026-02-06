---
description: Validate that the implementation meets the design — did we build the right thing? Produces a validation report.
argument-hint: Feature ID (e.g. 0001-auth-flow)
---

# Validate

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: validation
- consumes: design, tasks, review

## Params

- **feature-id**:
  - type: string
  - question: "Which feature? Provide the feature ID (e.g. 0001-auth-flow)"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.

## Agents

1. **cortex-team:tester** — Verify requirements are met, pass/fail criteria are satisfied, test coverage is adequate
2. **cortex-team:reviewer** — Verify the review findings were addressed and no regressions were introduced

Collaboration style: **sequential**. Tester validates against design requirements. Reviewer confirms review findings were resolved.

## Task

Validate feature "{{feature-id}}" against its design document.

This command answers two questions:
1. **Did we build it right?** (technical quality — review passed)
2. **Did we build the right thing?** (requirements — design fulfilled)

**Project context**: (inject loaded context memory)
**Design document**: (inject consumed design artifact)
**Task list**: (inject consumed tasks artifact)
**Review findings**: (inject consumed review artifact)

### Pre-check

Before starting validation, verify:
- The review artifact has `verdict: pass`. If not, tell the user to complete the review cycle first.
- All tasks in the task list are `done`. If not, tell the user to complete implementation first.

### Phase 1: Requirements Validation

Launch the **cortex-team:tester** with the design document and the implemented code:

For each requirement (REQ-NNN) in the design:
- Check if it's implemented
- Run or verify the corresponding tests (TEST-NNN)
- Check against the pass/fail criteria defined in the design
- Mark: met / not met / partially met

For each risk (RISK-NNN) in the design:
- Check if the mitigation was implemented
- Mark: mitigated / not mitigated / partially mitigated

### Phase 2: Review Resolution

Launch the **cortex-team:reviewer** to verify:
- Every critical finding from the review has been addressed
- No regressions were introduced by the fixes
- The review's warnings that were accepted are documented

### Phase 3: Produce Validation Report

Write the artifact to `.cortex/artifacts/{{feature-id}}.validation.md` with this EXACT structure:

```markdown
---
artifact: validation
feature: {{feature-slug}}
feature-id: {{feature-id}}
status: active
command: validate
created: YYYY-MM-DD
source:
  - {{feature-id}}.design.md
  - {{feature-id}}.tasks.md
  - {{feature-id}}.review.md
agents: [tester, reviewer]
verdict: pass/fail
---

# Validation: {{feature-name}}

Design: {{feature-id}}.design.md
Tasks: {{feature-id}}.tasks.md
Review: {{feature-id}}.review.md

## Verdict: PASS / FAIL

## Requirements Check

| Requirement | Status | Evidence |
|---|---|---|
| REQ-001: (description) | met/not met/partial | (file:line or test name) |
| REQ-002: ... | ... | ... |

## Test Results

| Test | Level | Status | Covers |
|---|---|---|---|
| TEST-001: (description) | unit/integration/e2e | pass/fail | REQ-NNN |
| TEST-002: ... | ... | ... | ... |

## Pass/Fail Criteria

| Criterion | Status | Evidence |
|---|---|---|
| (criterion from design) | pass/fail | (explanation) |

## Risk Mitigation

| Risk | Status | Evidence |
|---|---|---|
| RISK-001: (description) | mitigated/not mitigated | (how) |

## Review Resolution

| Finding | Status | Evidence |
|---|---|---|
| FINDING-001: (title) | resolved/unresolved | (what was done) |

## Summary
(overall assessment — is this feature ready for production?)

## Remaining Concerns
(anything the team should be aware of post-release)
```

If verdict is **fail**, tell the user which requirements/criteria were not met and recommend next steps.
