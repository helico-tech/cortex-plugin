---
description: Review implemented code — find bugs, convention violations, and structural issues. Feeds findings back as tasks if not passed.
argument-hint: Feature ID (e.g. 0001-auth-flow)
---

# Review

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: review
- consumes: tasks (optional), design (optional)

## Params

- **feature-id**:
  - type: string
  - question: "Which feature? Provide the feature ID (e.g. 0001-auth-flow)"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.

## Agents

1. **cortex-team:reviewer** — Find bugs, edge cases, convention violations, security issues
2. **cortex-team:architect** — Evaluate structural integrity against the design decisions

Collaboration style: **parallel then react then merge**. Both agents review independently, react to each other's summaries, then findings are merged and deduplicated.

## Task

Review the implementation of feature "{{feature-id}}".

**Project context**: (inject loaded context memory)
**Design document**: (inject consumed design artifact, or omit if not available)
**Task list**: (inject consumed tasks artifact — to know what was implemented and where, or omit if not available)

If no design artifact is loaded, skip the Design Compliance section in the review artifact and focus on code quality, bugs, and conventions.

### Phase 1: Code Review

Launch the **cortex-team:reviewer** and **cortex-team:architect** in parallel:

**Reviewer** focus:
- Bugs, logic errors, missed edge cases
- Security issues (injection, auth bypass, data exposure)
- Convention violations (naming, error handling, patterns)
- If `context/idioms.md` is loaded: check against IDIOM-NNN rules specifically — reference the standard ID in any finding (e.g., "violates IDIOM-003")
- Specific, line-level findings with concrete fix suggestions

**Architect** focus:
- If design artifact is loaded: Does the implementation match the design decisions? Are there structural deviations from the agreed architecture? Reference DEC-NNN in findings.
- If no design artifact is loaded: Evaluate structural quality based on project context and codebase conventions only. Skip DEC-NNN checks — there are no design decisions to compare against.
- If `context/architecture.md` is loaded: check against ARCH-NNN rules specifically — reference the standard ID in any finding (e.g., "violates ARCH-002")
- Are the integration points clean?
- Any new technical debt introduced?

### Phase 1b: Reaction Round

Run a **reaction round** (see cortex-runner Step 4). Reviewer and architect see each other's bullet-point summaries and respond:

- **Reviewer** reacts to architect's findings: Are any structural deviations also causing bugs? Do architectural concerns change the severity of code-level findings?
- **Architect** reacts to reviewer's findings: Do any bugs indicate a deeper structural problem? Are convention violations symptoms of a design mismatch?

This cross-reference catches findings that span both concerns — e.g., a bug that's also an architectural deviation, or a convention violation rooted in a structural shortcut.

### Phase 2: Merge & Classify Findings

Merge findings from both agents, including reaction round additions. Deduplicate. Classify each finding:

- **Critical**: Must fix. Bugs, security issues, design violations.
- **Warning**: Should fix. Convention violations, missing error handling, structural concerns.
- **Nit**: Take it or leave it. Style preferences, minor improvements.

### Phase 3: Verdict

Determine the verdict:
- **pass**: No critical issues, warnings are acceptable
- **fail**: Critical issues exist, must fix before validation

### Phase 4: Produce Review Artifact

Write the artifact to `.cortex/artifacts/{{feature-id}}.review.md` with this EXACT structure:

```markdown
---
artifact: review
feature: {{feature-slug}}
feature-id: {{feature-id}}
status: active
command: review
created: YYYY-MM-DD
source: {{feature-id}}.tasks.md
agents: [reviewer, architect]
verdict: pass/fail
---

# Review: {{feature-name}}

Design: {{feature-id}}.design.md (or "N/A — no design artifact" if not available)
Tasks: {{feature-id}}.tasks.md (or "N/A — no tasks artifact" if not available)

## Verdict: PASS / FAIL

## Critical Issues

### FINDING-001: (title)
- **Location**: `file:line`
- **Problem**: (specific description)
- **Fix**: (concrete suggestion)
- **Design ref**: (DEC-NNN or REQ-NNN if relevant)
- **Standard ref**: (ARCH-NNN or IDIOM-NNN if this violates a documented standard, omit if not applicable)

## Warnings

### FINDING-002: (title)
- **Location**: `file:line`
- **Problem**: (specific description)
- **Fix**: (concrete suggestion)

## Nits

### FINDING-003: (title)
- **Location**: `file:line`
- **Suggestion**: (improvement)

## Strengths
(what was done well — be specific)

## Design Compliance
(how well the implementation matches the design decisions — omit this entire section if no design artifact was loaded)
```

### Phase 5: Feedback Loop (on FAIL)

If verdict is **fail**:

**When tasks artifact exists:**
1. Convert each critical finding and selected warnings into new tasks
2. Append these tasks to `.cortex/artifacts/{{feature-id}}.tasks.md` with new TASK numbers continuing from the last
3. Set the new tasks' status to `pending`
4. Tell the user: "Review failed. New tasks have been added to the task list. Run `/cortex-team:implement {{feature-id}}` to address the findings."

**When no tasks artifact exists:**
1. List each critical finding and selected warning directly to the user with concrete fix instructions
2. Tell the user: "Review failed. Fix these issues and run `/cortex-team:review {{feature-id}}` again."
