---
description: Check codebase conformance to coding standards — report violations, discover undocumented patterns, propose standards evolution
argument-hint: Scope (file, module, directory, or 'everything')
---

# Conform

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: conformance
- consumes: none

## Params

- **conform-scope**:
  - type: string
  - question: "What area should be checked? (file path, directory, module name, or 'everything')"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **conform-focus**:
  - type: choice
  - question: "What should the focus be?"
  - options:
    - `both` — Check architecture standards and code idioms
    - `architecture` — Architecture only: module boundaries, dependency rules, layer separation
    - `idioms` — Code idioms only: naming, error handling, patterns, file structure

## Agents

1. **scout** — Scan codebase: find patterns, check files against standards, discover undocumented conventions
2. **architect** — Evaluate architectural constraints: module boundaries, dependency directions, layer violations
3. **pragmatist** — Challenge proposed new standards: is this intentional or coincidence? YAGNI?
4. **reviewer** — Compile structured compliance report, classify violations by severity

Collaboration style: **sequential with discovery**. Scout scans first. Architect evaluates structure. Reviewer compiles findings. Scout discovers undocumented patterns. Pragmatist challenges proposed additions.

## Task

Check conformance of "{{conform-scope}}" with focus on: {{conform-focus}}

**Project context**: (inject loaded context memory)

### Mode Detection

Check if `context/architecture.md` or `context/idioms.md` exist in the loaded context memory.

- If **neither exists**: run **Bootstrap Mode**
- If **at least one exists**: run **Compliance Mode**

---

### Bootstrap Mode (no standards files found)

No documented standards exist yet. Discover what the codebase actually does and propose initial standards.

#### Phase 1: Codebase Discovery

Launch the **scout**:
- Map folder structure: what are the top-level modules/directories, how is code organized?
- Analyze import/dependency patterns: who imports whom? Are there consistent directions?
- Find naming conventions: variables, functions, classes, files, directories
- Find error handling patterns: try-catch, Result types, error codes, null returns — what's dominant?
- Find test patterns: how are tests structured, what frameworks, what naming?
- Report raw findings with file:line references and frequency counts ("47 of 52 functions use pattern X")

#### Phase 2: Architecture Inference

Launch the **architect** with scout's findings (if {{conform-focus}} is `both` or `architecture`):
- Infer architectural layers from folder structure and import patterns
- Identify module boundaries: which directories are self-contained?
- Map dependency directions: which modules depend on which?
- Find boundary violations: imports that break the inferred pattern
- Propose ARCH-NNN standards with boundaries, rationale, and evidence

#### Phase 3: Idiom Extraction

Launch the **scout** again (if {{conform-focus}} is `both` or `idioms`):
- From the patterns found in Phase 1, identify candidates for codified idioms
- For each candidate: count how consistently it's followed (percentage)
- Only propose idioms followed in >70% of applicable cases — these are actual conventions, not aspirations
- Propose IDIOM-NNN standards with patterns, rationale, and evidence

#### Phase 4: Challenge

Launch the **pragmatist** with all proposed standards:
- For each proposed standard: is this an intentional design choice or an accident of history?
- Would codifying this actually help, or is it unnecessary ceremony?
- Are any proposed levels too strict (must vs should)?
- Remove any standards that don't earn their keep

#### Phase 5: Structure & Report

Launch the **reviewer**:
- Organize surviving proposals into the standard format (ID, level, rationale, boundary/pattern, examples)
- Group into architecture vs idioms
- Ensure IDs are sequential (ARCH-001, ARCH-002; IDIOM-001, IDIOM-002)

#### Phase 6: Produce Conformance Artifact

Write the artifact to `.cortex/artifacts/{feature-id}.conformance.md` with this EXACT structure:

```markdown
---
artifact: conformance
feature: {{conform-scope-slug}}
feature-id: {{feature-id}}
status: active
command: conform
created: YYYY-MM-DD
source: null
agents: [scout, architect, pragmatist, reviewer]
mode: bootstrap
---

# Conformance Report: {{conform-scope}} (Bootstrap)

## Discovery Summary
- Files scanned: (count)
- Architecture standards proposed: (count)
- Code idioms proposed: (count)
- Proposals rejected by pragmatist: (count)

## Proposed Architecture Standards

### ARCH-001: (title)
- **level**: must/should/may
- **rationale**: (why)
- **boundary**: (what depends on what, or what's forbidden)
- **evidence**: (what was found — counts, file references)
- **examples**: (good/bad code from the actual codebase)

### ARCH-002: ...

## Proposed Code Idioms

### IDIOM-001: (title)
- **level**: must/should/may
- **rationale**: (why)
- **pattern**: (what code should look like)
- **evidence**: (consistency percentage, file references)
- **examples**: (good/bad code from the actual codebase)

### IDIOM-002: ...

## Rejected Proposals
(standards the pragmatist killed, with reasoning)
```

In cortex-runner step 7, propose creating `context/architecture.md` and `context/idioms.md` from the accepted proposals. Strip the evidence fields — context files should be reference documents, not discovery reports.

---

### Compliance Mode (standards files exist)

Standards are documented. Check the codebase against them, find violations, and discover evolution opportunities.

#### Phase 1: Standards Compliance Scan

Launch the **scout** (if {{conform-focus}} is `both` or `idioms`):
- For each IDIOM-NNN in the loaded idioms standards: scan files in scope for violations
- Report each violation with file:line, the violated standard ID, and the specific code that violates it
- Count compliance rate per standard (e.g., "IDIOM-003: 45/50 files compliant, 5 violations")

Launch the **architect** in parallel (if {{conform-focus}} is `both` or `architecture`):
- For each ARCH-NNN in the loaded architecture standards: check module boundaries and dependencies
- Report each violation with file:line, the violated standard ID, and the offending import/dependency
- Check for new modules/directories that aren't covered by existing standards

#### Phase 2: Compile Violations

Launch the **reviewer** with all findings:
- Deduplicate violations
- Assign sequential VIOLATION-NNN IDs
- Classify by severity: `must`-level violations are critical, `should`-level are warnings, `may`-level are informational
- For each violation, provide a concrete fix suggestion

#### Phase 3: Discovery & Evolution

Launch the **scout** again:
- Look for consistent patterns NOT captured in existing standards
- Look for standards that no longer match reality (>30% violation rate suggests drift)
- Report undocumented patterns with frequency counts

Launch the **pragmatist** with scout's evolution findings:
- For proposed new standards: is this a real convention or noise?
- For drifted standards: should the code conform, or should the standard update?
- Challenge any proposed changes — don't update standards for trivialities

#### Phase 4: Produce Conformance Artifact

Write the artifact to `.cortex/artifacts/{feature-id}.conformance.md` with this EXACT structure:

```markdown
---
artifact: conformance
feature: {{conform-scope-slug}}
feature-id: {{feature-id}}
status: active
command: conform
created: YYYY-MM-DD
source: null
agents: [scout, architect, pragmatist, reviewer]
mode: compliance
---

# Conformance Report: {{conform-scope}}

## Compliance Summary

| Type | Standards Checked | Passed | Violations |
|---|---|---|---|
| Architecture | (count) | (count) | (count) |
| Code Idioms | (count) | (count) | (count) |

## Compliance by Standard

| Standard | Level | Compliance Rate | Violations |
|---|---|---|---|
| ARCH-001 | must | 95% | 2 |
| IDIOM-001 | should | 100% | 0 |
| ... | ... | ... | ... |

## Violations

### VIOLATION-001: (title)
- **standard**: ARCH-NNN or IDIOM-NNN
- **level**: must/should/may
- **location**: `file:line`
- **description**: (what's wrong)
- **suggestion**: (how to fix)

### VIOLATION-002: ...

## Proposed Standard Changes

### NEW: IDIOM-NNN (proposed)
- **level**: must/should/may
- **pattern**: (what the standard would be)
- **evidence**: (what was found in the codebase)
- **challenged**: (pragmatist's assessment)

### UPDATE: ARCH-NNN (revision proposed)
- **current**: (current rule text)
- **proposed**: (new rule text)
- **evidence**: (why it should change — drift percentage, examples)
- **rationale**: (justification)

### No changes proposed
(if no evolution needed, state this explicitly)
```

In cortex-runner step 7, propose updates to `context/architecture.md` and/or `context/idioms.md` based on the accepted proposals. Only propose changes that survived the pragmatist's challenge.
