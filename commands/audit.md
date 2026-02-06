---
description: Audit a section of the codebase — produce a health report covering quality, structure, test coverage, security, and tech debt
argument-hint: Area to audit (file, module, directory, or 'everything')
---

# Audit

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: audit
- consumes: none

## Params

- **audit-scope**:
  - type: string
  - question: "What area should be audited? (file path, directory, module name, or 'everything')"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **audit-focus**:
  - type: choice
  - question: "What should the audit focus on?"
  - options:
    - `all` — Full audit: quality, structure, tests, security, tech debt
    - `quality` — Code quality: naming, duplication, complexity, readability
    - `structure` — Architecture: coupling, cohesion, dependencies, patterns
    - `test-coverage` — Testing: gaps, missing edge cases, flaky areas
    - `security` — Security: input validation, auth, data exposure, dependencies

## Agents

1. **cortex-team:scout** — Map the area: files, dependencies, size, test coverage, conventions
2. **cortex-team:reviewer** — Find code quality issues, bugs, security concerns
3. **cortex-team:architect** — Evaluate structural integrity: coupling, cohesion, patterns, layering
4. **cortex-team:tester** — Assess test coverage: what's tested, what's missing, what's flaky

Collaboration style: **parallel assessment, then merge**. Scout maps first. Then reviewer, architect, and tester assess in parallel (each from their own angle). Findings are merged, deduplicated, and prioritized.

## Task

Audit "{{audit-scope}}" with focus on: {{audit-focus}}

**Project context**: (inject loaded context memory)

### Phase 1: Mapping

Launch the **cortex-team:scout**:
- Map every file in scope: size, age, change frequency (from git log)
- Identify dependencies within and outside the scope
- Report existing test coverage (which files have tests, which don't)
- Find conventions and patterns used in and around the scope
- Report raw numbers: file count, total lines, test-to-code ratio

### Phase 2: Parallel Assessment

Launch the following agents in parallel, each focused on **{{audit-focus}}** (or all areas if `all`):

**Reviewer** (quality + security):
- Code quality: naming, duplication, dead code, complexity hotspots
- Security: input validation, auth checks, data exposure, dependency vulnerabilities
- Specific, file:line-level findings

**Architect** (structure):
- Coupling between modules: who depends on whom, circular dependencies
- Cohesion within modules: does each module have a single responsibility?
- Pattern consistency: are the same patterns used throughout?
- Layering: are boundaries respected?

**Tester** (test coverage):
- Which code paths lack tests?
- Which tests are brittle or testing implementation details?
- What edge cases are missing?
- Where would a regression most likely slip through?

### Phase 3: Merge & Prioritize

Merge all findings. Deduplicate. Classify each:

- **Critical**: Active risk — bugs, security holes, untested critical paths
- **Warning**: Accumulating debt — growing complexity, missing tests, coupling
- **Info**: Worth knowing — patterns, conventions, statistics

Order by impact within each category.

### Phase 4: Produce Audit Report

Write the artifact to `.cortex/artifacts/{feature-id}.audit.md` with this EXACT structure:

```markdown
---
artifact: audit
feature: {{audit-scope-slug}}
feature-id: {{feature-id}}
status: active
command: audit
created: YYYY-MM-DD
source: null
agents: [scout, reviewer, architect, tester]
---

# Audit: {{audit-scope}}

Focus: {{audit-focus}}

## Overview

| Metric | Value |
|---|---|
| Files in scope | (count) |
| Total lines | (count) |
| Test files | (count) |
| Test-to-code ratio | (percentage) |
| Dependencies (internal) | (count) |
| Dependencies (external) | (count) |

## Health Summary

| Area | Rating | Key Concern |
|---|---|---|
| Code Quality | good/fair/poor | (one-line summary) |
| Structure | good/fair/poor | (one-line summary) |
| Test Coverage | good/fair/poor | (one-line summary) |
| Security | good/fair/poor | (one-line summary) |

## Critical Findings

### AUDIT-001: (title)
- **Area**: quality/structure/test-coverage/security
- **Location**: `file:line`
- **Finding**: (specific description)
- **Impact**: (what can go wrong)
- **Recommendation**: (what to do about it)

### AUDIT-002: ...

## Warnings

### AUDIT-003: (title)
- **Area**: quality/structure/test-coverage/security
- **Location**: `file:line` or module-level
- **Finding**: (specific description)
- **Impact**: (what could accumulate)
- **Recommendation**: (what to do about it)

### AUDIT-004: ...

## Informational

### AUDIT-005: (title)
- **Area**: quality/structure/test-coverage/security
- **Detail**: (observation)

## Dependency Map

(text visualization of key dependencies within the audited scope)

## Test Coverage Map

| Area | Coverage | Gaps |
|---|---|---|
| (module/file) | good/partial/none | (what's missing) |

## Tech Debt Inventory

| Item | Severity | Effort to Fix | Location |
|---|---|---|---|
| (description) | high/medium/low | small/medium/large | `file:line` |

## Recommendations

(ordered list of suggested next actions, highest impact first)
```
