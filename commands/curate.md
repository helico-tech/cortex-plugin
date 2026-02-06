---
description: Distill journal entries into context updates — bulk-process execution history into curated project knowledge
argument-hint: "Optional: date range like '2026-02' or 'last-month'"
---

# Curate

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: curation
- consumes: none

## Params

- **since**:
  - type: string
  - question: "Curate journals since when? (YYYY-MM-DD, YYYY-MM, 'last-month', or 'all')"
  - required: true
  - default: all
  - Use $ARGUMENTS if provided, otherwise ask.
- **focus**:
  - type: choice
  - question: "What kind of knowledge should we focus on extracting?"
  - options:
    - `all` — Everything: conventions, domain knowledge, decisions, patterns
    - `conventions` — Coding standards, naming patterns, tool choices, workflow norms
    - `domain` — Domain model terms, business rules, entity relationships
    - `decisions` — Architectural and design decisions with rationale

## Agents

1. **cortex-team:scout** — Read all journal entries in range, read existing context files, extract raw findings
2. **cortex-team:pragmatist** — Challenge every finding: is this worth remembering? Is it already captured? Will it be useful?
3. **cortex-team:writer** — Draft concrete edits to context files from the surviving findings

Collaboration style: **sequential pipeline**. Scout extracts, pragmatist filters, writer synthesizes. Each stage narrows the funnel.

## Task

Curate journal entries into context updates. Focus: {{focus}}

**Project context**: (inject loaded context memory)

### Phase 1: Extract

Launch the **cortex-team:scout**:
- Read all `.md` files in `.cortex/memory/journal/` matching the date filter "{{since}}"
  - `all` → read everything
  - `YYYY-MM` → read that month's directory
  - `YYYY-MM-DD` → read from that date forward
  - `last-month` → read the previous calendar month
- Read all existing context files from `.cortex/memory/context/`
- For each journal entry, extract:
  - Key findings
  - Decisions made (with rationale)
  - Patterns observed
  - Domain terms introduced
  - Conventions established or reinforced
- Cross-reference with existing context: what's already captured vs what's new
- Report: total journals read, total findings extracted, how many are already in context

### Phase 2: Filter

Launch the **cortex-team:pragmatist** with scout's findings:
- For each finding, challenge it:
  - Is this a **pattern** (appeared multiple times) or a **one-off**? One-offs get cut.
  - Is this **already captured** in existing context files? Duplicates get cut.
  - Will this **actually help** a future session? Trivia gets cut.
  - Is this **still true**? Contradicted findings get flagged.
- Classify surviving findings:
  - **Add**: new knowledge not in any context file
  - **Update**: refines or corrects something already in context
  - **Contradiction**: conflicts with existing context (flag for human decision)
- Be ruthless. Context that grows without bound defeats the purpose. If in doubt, cut it.

### Phase 3: Synthesize

Launch the **cortex-team:writer** with the filtered findings:
- For each **Add** finding, draft the exact text to append to the appropriate context file
  - If no appropriate file exists, propose a new context file with a clear name
  - Keep entries concise — one bullet point or short paragraph per finding
- For each **Update** finding, draft the exact replacement text
- For each **Contradiction**, present both sides and let the human decide
- Match the voice and format of existing context files — don't introduce a new style

### Phase 4: Produce Curation Report

Write the artifact to `.cortex/artifacts/{feature-id}.curation.md` with this EXACT structure:

```markdown
---
artifact: curation
feature: curation-{{date-range-slug}}
feature-id: {{feature-id}}
status: active
command: curate
created: YYYY-MM-DD
source: null
agents: [scout, pragmatist, writer]
---

# Curation Report

Period: {{since}}
Focus: {{focus}}

## Journals Analyzed

| Period | Count |
|---|---|
| (month) | (count) |
| **Total** | **(total)** |

## Findings Summary

| Category | Extracted | Filtered Out | Surviving |
|---|---|---|---|
| Conventions | (n) | (n) | (n) |
| Domain knowledge | (n) | (n) | (n) |
| Decisions | (n) | (n) | (n) |
| Patterns | (n) | (n) | (n) |
| **Total** | **(n)** | **(n)** | **(n)** |

## Proposed Context Updates

### Additions

#### ADD-001: (title)
- **Target file**: `.cortex/memory/context/(filename).md`
- **Content**:
  > (exact text to add)
- **Evidence**: (journal entries that support this — filenames)

#### ADD-002: ...

### Updates

#### UPD-001: (title)
- **Target file**: `.cortex/memory/context/(filename).md`
- **Current text**:
  > (what's there now)
- **Proposed text**:
  > (what it should say)
- **Reason**: (why the change)
- **Evidence**: (journal entries)

#### UPD-002: ...

### Contradictions

#### CON-001: (title)
- **Existing context**: (what context currently says)
- **Journal evidence**: (what journals suggest)
- **Decision needed**: (what the human should resolve)

## Rejected Findings

| Finding | Reason |
|---|---|
| (description) | one-off / already captured / trivia / outdated |
```

**IMPORTANT**: After producing the artifact, step 7 (propose context updates) should use the **Proposed Context Updates** section from this artifact as its proposals. The curation report IS the input to step 7 — present the additions, updates, and contradictions to the user for approval. This is the whole point of the command.
