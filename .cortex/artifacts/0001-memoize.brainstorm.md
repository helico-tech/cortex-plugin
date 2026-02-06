---
artifact: brainstorm
feature: memoize
feature-id: "0001"
status: active
command: brainstorm
created: 2026-02-06
source: null
agents: [scout, pragmatist, architect]
---

# Brainstorm: Memoize Command

## Outcome

Developers have thoughts, insights, and realizations between command runs that get lost. The existing memory system captures knowledge as a byproduct of command execution (step 7 context proposals) and through batch curation (`curate` command), but there's no path for ad-hoc thoughts that happen outside command runs.

`memoize` is a lightweight, guided capture-to-journal command. It does NOT write directly to context — the thought goes to journal, and `curate` later promotes it to context if worthy. This preserves the existing quality wall where context stays small, curated, and human-approved. The developer picks a category (Fact, Convention, Insight, Correction), a scout checks existing memory for contradictions and overlaps, asks 1-2 clarifying questions, and a writer sharpens the thought into a clean journal entry.

The ceremony is adaptive to the weight of the thought but always lightweight — the goal is under 30 seconds for a simple fact, under 2 minutes for a complex insight. No artifact is produced. No context proposal is made. The journal is the output. Curate handles promotion later.

### Suggested Feature Name
memoize

### Suggested Feature Description
A guided command for capturing ad-hoc developer thoughts into journal entries. Developer states a thought, picks a category (Fact/Convention/Insight/Correction), scout validates against existing memory, writer sharpens the entry. Writes to journal for later curation — does not write directly to context.

## Ideas Explored

### IDEA-001: Direct-to-context memory writer
- **Status**: discarded
- **Description**: A command that writes directly to `.cortex/memory/context/` files, with full agent ceremony (scout, pragmatist, architect, writer) and multi-tier adaptive ceremony based on thought complexity.
- **Arguments for**: Immediate impact on memory, no waiting for curation cycle.
- **Arguments against**: Bypasses the quality wall that the entire memory system is built on. Design doc explicitly states "Agent-written memory that auto-loads is a bloat machine." Creates a pipe that works against the memory architecture's core invariant.
- **Agent perspectives**: Pragmatist argued forcefully against this — the system already has two memory paths (step 7 and curate), and a third direct-write path undermines the deliberate design.

### IDEA-002: Four-category adaptive ceremony with full agent teams
- **Status**: discarded
- **Description**: Four categories (Fact, Convention, Insight, Correction) each triggering different levels of ceremony — Facts get lightweight treatment, Insights get full multi-agent treatment with architect and pragmatist.
- **Arguments for**: Matches effort to thought weight. Complex insights deserve more scrutiny.
- **Arguments against**: Over-engineered for what is fundamentally a capture operation. The categories are useful as routing metadata for curate, but driving different ceremony levels from them adds complexity without proportional value. Four agent spawns to write a bullet point.
- **Agent perspectives**: Pragmatist called this "ceremony for ceremony's sake." Architect agreed the categories have value as frontmatter metadata but not as ceremony drivers.

### IDEA-003: Capture-to-journal with light sharpening
- **Status**: adopted
- **Description**: Write to journal (not context), with scout checking existing memory and writer sharpening the prose. Categories used as triage tags for curate, not ceremony drivers. Two agents, sequential, under 30 seconds for simple entries.
- **Arguments for**: Fits existing memory architecture perfectly — journal is the raw tier, context is curated. Preserves quality wall. Lightweight enough to actually use. Categories give curate routing metadata without adding intake friction.
- **Arguments against**: Adds a step between thought and context (must wait for curate). But this is a feature, not a bug.
- **Agent perspectives**: Architect validated the structural fit — journal path, cortex-runner usage, curate integration all have clean answers. Pragmatist conceded this avoids the "door in the wall" problem.

### IDEA-004: Zero sharpening / raw dump
- **Status**: discarded
- **Description**: Just write the developer's words verbatim to journal. No agents, no questions.
- **Arguments for**: Maximum speed, minimum friction.
- **Arguments against**: Risk of cryptic notes ("check the thing with the tokens") that are useless a week later. Light sharpening (1-2 clarifying Qs) is a small cost for dramatically better recall.

### IDEA-005: Sharpening on request
- **Status**: discarded
- **Description**: Capture verbatim by default, let developer say "sharpen this" to pull in agents.
- **Arguments for**: Best of both worlds — fast when you want fast, deep when you want deep.
- **Arguments against**: Adds command complexity for an edge case. If the default is always light sharpening (1-2 questions), the overhead is negligible and the quality floor is always decent.

## Key Decisions

- BDEC-001: **Journal, not context** — Memoize writes to journal only. Context promotion happens via curate. This preserves the quality wall and fits the existing memory architecture.
- BDEC-002: **Developer picks category explicitly** — Four categories (Fact, Convention, Insight, Correction) selected by the developer, not inferred. Transparent, no misclassification risk. Categories serve as routing metadata for curate.
- BDEC-003: **Light sharpening always** — Scout asks 1-2 clarifying questions, writer cleans up prose. No adaptive ceremony — every thought gets the same lightweight treatment.
- BDEC-004: **Contradiction: block and show. Overlap: merge suggestion** — When a thought contradicts existing memory, flag it and ask "are you sure?" When it overlaps, propose a merged version. Different responses for different conflict types.
- BDEC-005: **Cortex-runner, skip steps 3/5/7** — Uses cortex-runner for param collection, memory loading, and journal writing. Skips artifact consumption, artifact production, and context proposals. Follows precedent set by `rationale` command.
- BDEC-006: **Scout checks, writer sharpens** — Two agents, sequential. Scout reports contradictions/overlaps and asks clarifying questions. Writer produces the clean journal entry. Respects agent role boundaries.
- BDEC-007: **Journal path uses `memo` slug** — Path: `YYYY-MM/YYYY-MM-DD-memo-{slug}.md`. The `memo` token replaces the feature-id position, keeping entries in the same directory curate already scans.
- BDEC-008: **`memo-category` frontmatter for curate** — Journal entries include `memo-category: fact|convention|insight|correction` so curate can treat human-captured thoughts as higher-signal than auto-generated findings.

## Open Questions

- Should curate be updated now to recognize `memo-category` frontmatter, or defer until memoize has been used enough to validate the categories?
- What are the exact clarifying questions per category? (e.g., for Fact: "what part of the codebase does this relate to?", for Correction: "what does the current memory say that's wrong?")
- Should there be a `--quick` flag that skips clarifying questions for when you really just need to dump a thought?
- How should the slug be generated — from the developer's words, or should the writer propose one?

## Discarded Paths

| Path | Reason Discarded |
|---|---|
| Direct-to-context writer | Bypasses quality wall, contradicts memory design philosophy |
| Four-tier adaptive ceremony | Over-engineered, categories useful as metadata not ceremony drivers |
| Zero sharpening / raw dump | Risk of cryptic, useless notes |
| Sharpening on request | Adds complexity for marginal benefit over always-light-sharpening |
| Standalone (no cortex-runner) | Journal format drift risk, reimplements shared ceremony |
| Mini-runner skill | YAGNI — only two commands would use it |
| Separate memo directory | Fractures journal, forces curate to scan two locations |
| Writer does contradiction checking | Erodes agent role boundaries |

## Suggested Next Step

Run: `/cortex-team:design memoize`

With this context already explored, the design command can skip initial research and focus on the command definition, agent instructions, clarifying question design, and curate integration details.
