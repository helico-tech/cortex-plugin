---
artifact: design
feature: memoize
feature-id: 0001-memoize
status: active
command: design
created: 2026-02-06
source: 0001-memoize.brainstorm.md
agents: [architect, pragmatist, tester]
---

# Design: memoize

## Problem Statement

Developers accumulate knowledge during coding sessions — conventions they notice, corrections to their mental model, facts about the domain, insights about patterns — that currently has nowhere to go between command runs. The memory system captures knowledge as a byproduct of command execution (step 7 context proposals) and through batch curation (`curate`), but there is no path for ad-hoc thoughts that happen outside command runs.

Memoize fills this gap: a fast, minimal "write this down" command that deposits a thought into the journal for `curate` to promote to context later. It does NOT write directly to context — preserving the quality wall that keeps context small, curated, and human-approved.

## Requirements

- REQ-001: Developer can capture a freeform thought into the journal system | priority: must
- REQ-002: Writer agent sharpens the thought (grammar, clarity, specificity) without changing meaning | priority: must
- REQ-003: Writer scans loaded memory for obvious contradictions or overlaps and notes them inline | priority: must
- REQ-004: Command never writes to `.cortex/memory/context/` — journal only | priority: must
- REQ-005: Command never produces an artifact to `.cortex/artifacts/` | priority: must
- REQ-006: Journal entry is compatible with curate's processing pipeline | priority: must
- REQ-007: Journal entry uses the path pattern `YYYY-MM/YYYY-MM-DD-memo-{slug}.md` | priority: must
- REQ-008: Writer generates a 2-4 word slug from the sharpened content | priority: must
- REQ-009: Command creates the `YYYY-MM/` directory if it doesn't exist | priority: must
- REQ-010: Thought can be provided via `$ARGUMENTS` or interactive prompt | priority: must
- REQ-011: Command completes in under 10 seconds for typical inputs | priority: should
- REQ-012: Sharpened journal entry body is 3-8 lines | priority: should

## Decisions

### DEC-001: One agent (writer), not two (scout + writer)

- **Decided**: Single writer agent handles everything — memory scan, sharpening, slug generation, journal write.
- **Rationale**: The "memory scan" is reading 5-10 small markdown files already loaded by cortex-runner step 2, not codebase investigation. The scout's role (codebase mapping) is overkill. One agent means one spawn, half the latency, under 10 seconds.
- **Alternatives rejected**: Two-agent sequential (scout + writer) — adds ~15 seconds latency for a probabilistic contradiction check. The writer can do a lightweight version of the same scan.

### DEC-002: No category param

- **Decided**: No `memo-category` choice param. One param only: `thought`.
- **Rationale**: Nothing reads categories today. Curate has its own classification taxonomy that doesn't match. The categories are a premature taxonomy designed before a corpus exists. YAGNI — add after 50 memos prove it's needed.
- **Alternatives rejected**: Four categories (fact/convention/insight/correction) as routing metadata for curate — adds friction for zero current value. Backfilling categories later is a simple grep + writer pass if ever needed.

### DEC-003: No clarifying questions

- **Decided**: Zero questions beyond the initial thought. Writer sharpens what it gets.
- **Rationale**: Clarifying questions often ask the user to repeat what they already stated in their thought. Every question is a chance to abandon the command. A vague memo is better than no memo. The writer can extract context from the thought itself.
- **Alternatives rejected**: 2 category-specific clarifying questions — adds friction, scout-phase dependency, and user interview behavior that doesn't match agent role definitions.

### DEC-004: Contradictions are non-blocking, noted inline

- **Decided**: When the writer spots an obvious contradiction with loaded memory, it appends a `### Note` section to the journal entry. It does NOT block execution or ask the user to resolve.
- **Rationale**: The contradiction scan is probabilistic (keyword matching across unstructured markdown). Blocking on a probabilistic check makes the command feel unpredictable. Noting it for curate is enough — curate has the full pipeline (scout + pragmatist + writer) to handle resolution properly.
- **Alternatives rejected**: Block-and-show with resolution options (from brainstorm BDEC-004) — required a scout agent and user interaction mid-flow. Too heavy for a quick-deposit command.

### DEC-005: Cortex-runner skip pattern — steps 3, 5, 7

- **Decided**: Use cortex-runner. Execute steps 1 (params), 2 (load memory), 4 (execute writer), 6 (journal write). Skip steps 3 (no consumed artifacts), 5 (no produced artifact), 7 (no context proposals).
- **Rationale**: Cortex-runner provides param collection, memory loading, and journal writing for free. The skip pattern follows the precedent set by `rationale` (which skips 5-7). Memoize differs from rationale by keeping step 6 — the journal entry IS the output.
- **Alternatives rejected**: Standalone command bypassing cortex-runner — reimplements shared ceremony, risks journal format drift. Mini-runner skill — YAGNI, only two commands would use it.

### DEC-006: Journal path uses `memo-{slug}` in place of feature-id

- **Decided**: Path: `.cortex/memory/journal/YYYY-MM/YYYY-MM-DD-memo-{slug}.md`. The `memo-` token replaces the feature-id position in the standard journal naming convention.
- **Rationale**: Memos are not tied to features. Using `feature-id: none` in frontmatter and `memo-{slug}` in the filename keeps memo journals scannable and distinguishable from command journals in the same directory.
- **Alternatives rejected**: Synthetic `0000-memo` feature-id (collision risk, magic number), separate memo directory (fractures journal, forces curate to scan two locations).

### DEC-007: Writer generates slug

- **Decided**: Writer generates a 2-4 word lowercase hyphenated slug from the sharpened content. No user confirmation.
- **Rationale**: The slug is a filename component, not a user-facing label. The writer sees the sharpened content and picks a better slug than the user could from raw input. Asking for a slug adds unnecessary interaction.
- **Alternatives rejected**: User-provided slug — adds a question to an already-minimal flow.

### DEC-008: Curate changes deferred

- **Decided**: No changes to the curate command. Curate already reads all journal entries and classifies findings.
- **Rationale**: Curate processes memo journals the same as any other journal entry. The `feature-id: none` and `command: memoize` frontmatter fields are enough for curate to identify memos if needed later. Add category-aware routing when memos actually exist and curate's filtering proves insufficient.
- **Alternatives rejected**: Immediate curate update with memo-category routing — premature, no memos exist to route yet.

### DEC-009: Step 6 journal write — writer writes during step 4, step 6 confirms

- **Decided**: The writer writes the journal file as part of its task execution (step 4). Step 6 of cortex-runner verifies the journal exists and is well-formed rather than writing a second entry.
- **Rationale**: The writer needs to write the file to generate the slug-based filename and fill in the sharpened content. Having cortex-runner step 6 also write would produce a duplicate or conflict. The writer owns the write; step 6 validates.
- **Alternatives rejected**: Step 6 writes journal from writer's output — would require the writer to return structured data instead of writing directly, adding complexity for no benefit.

## Implementation Approach

### Components

**Command file**: `commands/memoize.md`
- Frontmatter: description, argument-hint
- Artifacts: produces none, consumes none
- Params: `thought` (string, required, from $ARGUMENTS)
- Agents: writer only
- Task: single-phase writer execution
- Skip declaration: steps 3, 5, 7

**No other files need to be created or modified.** The writer agent already exists. Cortex-runner already supports step skipping. Curate already reads all journals.

### Data Flow

```
User thought (text)
  → cortex-runner step 1: collect param
  → cortex-runner step 2: load .cortex/memory/context/*.md
  → cortex-runner step 4: spawn writer agent
    → writer reads loaded memory
    → writer scans for contradictions/overlaps
    → writer sharpens thought (grammar, clarity, specificity)
    → writer generates slug
    → writer writes journal file to .cortex/memory/journal/YYYY-MM/YYYY-MM-DD-memo-{slug}.md
    → writer shows confirmation to user
  → cortex-runner step 6: verify journal exists
  → done
```

### Files to Create/Modify

| Action | File | Description |
|---|---|---|
| **Create** | `commands/memoize.md` | The command definition |

That's it. One file.

### Integration Points

- **cortex-runner** (`skills/cortex-runner/SKILL.md`): Steps 1, 2, 4, 6. Skip pattern follows `rationale.md` precedent.
- **writer agent** (`agents/writer.md`): Used as-is. No changes needed — writer already has Read, Write, Edit, Grep, Glob, LS tools.
- **curate command** (`commands/curate.md`): Curate reads all `.cortex/memory/journal/**/*.md`. Memo entries will be picked up automatically. No curate changes needed.
- **Journal directory** (`.cortex/memory/journal/YYYY-MM/`): Writer must create the `YYYY-MM/` subdirectory if it doesn't exist.

### Journal Entry Format

```markdown
---
date: YYYY-MM-DD
command: memoize
feature-id: none
agents: [writer]
artifact-produced: none
---

## Memo: (sharpened title — 5-10 words)

(Sharpened thought — 3-8 lines. The core knowledge being preserved.)
```

If contradictions or overlaps found, append:

```markdown
### Note

Potentially conflicts with (file:section). Consider reviewing during curate.
```

### Writer Agent Instructions (for command Task section)

The writer receives:
- The user's raw thought
- All loaded context memory (from cortex-runner step 2)

The writer does:
1. **Scan memory** — Quick read of loaded context for obvious contradictions or overlaps. Keyword-level, not deep semantic analysis. If found, note for the journal entry.
2. **Sharpen** — Fix grammar, make specific, keep short. Do NOT change meaning. Do NOT editorialize or add analysis the user didn't provide. 3-8 lines.
3. **Generate slug** — 2-4 words, lowercase, hyphenated, derived from the sharpened thought's core concept.
4. **Write journal** — Create `YYYY-MM/` directory if needed. Write the file. Use the exact template.
5. **Confirm** — Show the user: file path, sharpened content, and any contradiction/overlap notes. Suggest running `/cortex-team:curate` when they have a batch of memos to promote.

## Test Strategy

### Risk Areas

| Risk | Severity | Likelihood | Mitigation |
|---|---|---|---|
| RISK-001: Malformed frontmatter breaks curate pipeline | high | medium | Validate frontmatter fields match curate's expected format |
| RISK-002: Command writes to context instead of journal | high | low | Writer instructions explicitly prohibit context writes; validate post-run |
| RISK-003: Cortex-runner steps 3/5/7 execute when skipped | high | medium | Skip declaration in command file; verify no artifact/context prompts |
| RISK-004: Directory creation failure on first memo | medium | medium | Writer instructions include directory creation; test with empty journal |
| RISK-005: Slug collision overwrites previous memo | medium | low | Writer should check for existing file and append disambiguator if needed |
| RISK-006: Sharpening alters developer's intended meaning | medium | medium | Writer instructions emphasize "do NOT change meaning"; test with edge cases |
| RISK-007: Contradiction scan misses obvious conflict | low | high | Probabilistic by design; curate catches it later. Not a blocking failure. |
| RISK-008: Curate cannot process memo journal entries | high | low | `feature-id: none` is non-standard; verify curate handles it |

### Test Plan

- TEST-001: Happy path with empty memory — memo created, no errors | level: e2e | covers: REQ-001, REQ-007, REQ-009
- TEST-002: Happy path with existing context — context loaded, no false contradictions | level: e2e | covers: REQ-003
- TEST-003: Contradiction detection — obvious conflict noted in journal | level: e2e | covers: REQ-003
- TEST-004: Overlap detection — duplicate noted, not blocked | level: e2e | covers: REQ-003
- TEST-005: $ARGUMENTS input path — thought from args, no re-prompt | level: e2e | covers: REQ-010
- TEST-006: Interactive input path — prompted when no args | level: e2e | covers: REQ-010
- TEST-007: Directory creation — YYYY-MM/ created on first memo | level: e2e | covers: REQ-009
- TEST-008: Second memo same month — no collision, no overwrites | level: e2e | covers: REQ-007
- TEST-009: Slug uniqueness — similar thoughts produce different slugs | level: e2e | covers: REQ-008, RISK-005
- TEST-010: Long input sharpened — 500+ words → 3-8 lines | level: e2e | covers: REQ-002, REQ-012
- TEST-011: Short input handled — 2 words → valid entry | level: e2e | covers: REQ-002
- TEST-012: Step skip verification — steps 3/5/7 not executed | level: e2e | covers: REQ-004, REQ-005
- TEST-013: Frontmatter curate compatibility — all fields valid | level: e2e | covers: REQ-006
- TEST-014: Memoize → curate roundtrip — curate processes memo | level: integration | covers: REQ-006

### Edge Cases

- Thought containing markdown formatting (bold, code blocks, links) — preserved in body
- Thought containing YAML-unsafe characters (colons, quotes) — body only, not frontmatter
- Very short input ("use zod") — writer produces valid entry from minimal input
- No `.cortex/` directory at all — requires `cortex-init` first, error with guidance

### Pass/Fail Criteria

1. Journal file exists at the correct path pattern
2. Frontmatter contains all required fields with correct values
3. Body is 3-8 lines with sharpened content
4. No files written to `.cortex/memory/context/` or `.cortex/artifacts/`
5. No context update prompt shown to user
6. Curate can read and extract findings from the memo journal entry

## Open Questions

- BC-001: Should memoize create `.cortex/memory/journal/` if it doesn't exist, or require `cortex-init`? Recommend: require `cortex-init` — other commands also depend on the directory structure.
