# Standards Enforcement Design

## Problem

Agentic coding requires clear structures. Without documented and enforced coding standards, agents drift from project conventions, write code that doesn't match idioms, and violate architectural boundaries. The current system is guidance-based (freeform conventions.md, agents told to "match existing patterns") — not enforcement-based. Violations are caught post-hoc in review, not prevented during implementation.

## Goals

1. **Discover** — Extract coding idioms and architectural constraints from an existing codebase
2. **Document** — Structured, machine-readable standards with IDs, severity levels, rationale, and examples
3. **Enforce** — Check compliance during development (design time for architecture, implement time for idioms)
4. **Evolve** — Standards automatically improve as the codebase evolves; drift and gaps are detected and proposed as updates

## Key Decisions

### Two types of standards (different beasts)

- **Code idioms** — local, per-file, pattern-matchable. Naming conventions, error handling, file structure. Checked by looking at a single file or function.
- **Architectural constraints** — cross-cutting, relational, structural. Module boundaries, dependency directions, layer separation. Require understanding relationships between modules.

Different documentation structure, different enforcement surface.

### Standards live in context/ (not artifacts)

Standards are living reference documents, not lifecycle artifacts. They live in `context/idioms.md` and `context/architecture.md` — always loaded by cortex-runner step 2, human-approved via step 7.

Journal entries track provenance (when standards were discovered, changed, validated). No artifact-level ceremony needed.

### `conform` is the core primitive

A standalone command that scans the codebase against standards, reports compliance, and proposes evolution. No consumed artifacts — reads context files and the codebase directly. Runnable anytime.

Adapts behavior: bootstrap mode (no standards exist yet) discovers and proposes initial standards. Compliance mode (standards exist) checks violations and proposes evolution.

### Enforcement woven into existing flow

- `design` — architect verifies proposed design against ARCH-NNN rules before finalizing
- `implement` — scout checks changed files against idioms and architecture per-task
- `review` — findings reference specific standard IDs (ARCH-NNN, IDIOM-NNN)
- `tidy` — includes should-level idiom violations in find phase

### Evolution via conform

Conform handles the full evolution loop:
- Discovers undocumented patterns and proposes new standards
- Detects drift (standard no longer matches reality) and proposes updates
- Feeds into cortex-runner step 7 for human approval

## Standards File Structure

### context/architecture.md

```markdown
# Architecture Standards

## ARCH-001: (constraint title)
- **level**: must/should/may
- **rationale**: (why this constraint exists)
- **boundary**: (what imports/depends on what, or what is forbidden)
- **examples**: (good/bad code snippets)
```

### context/idioms.md

```markdown
# Code Idioms

## IDIOM-001: (idiom title)
- **level**: must/should/may
- **rationale**: (why this idiom exists)
- **pattern**: (what the code should look like)
- **examples**: (good/bad code snippets)
```

## Changes Made

### New: `conform` command
- Standalone standards compliance command
- Bootstrap mode: discovers patterns, proposes initial standards
- Compliance mode: checks violations, proposes evolution
- Agents: scout, architect, pragmatist, reviewer
- Produces: conformance artifact

### Enhanced: `design` command
- Phase 2b: architect checks design against ARCH-NNN rules before finalizing

### Enhanced: `implement` command
- Step 3b: scout checks changed files against idioms and architecture per-task

### Enhanced: `review` command
- Findings reference standard IDs (ARCH-NNN, IDIOM-NNN)
- Added `Standard ref` field to finding template

### Enhanced: `tidy` command
- Added `standards` focus option
- Loads idioms when focus is `all` or `standards`

### Updated: `maintenance` workflow
- `audit → conform → tidy → curate` (conform inserted between audit and tidy)

### Updated: cortex-runner
- Added VIOLATION-NNN, ARCH-NNN, IDIOM-NNN numbering schemes
