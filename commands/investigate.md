---
description: Investigate a topic — explore the codebase and research online to gather facts before making decisions
argument-hint: What to investigate (feature area, technology, problem)
---

# Investigate

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: investigation
- consumes: none

## Params

- **investigation-topic**:
  - type: string
  - question: "What do you want to investigate? (feature area, technology choice, problem, pattern, etc.)"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **investigation-focus**:
  - type: choice
  - question: "Where should the investigation focus?"
  - options:
    - `codebase` — Focus on what exists in the codebase: patterns, structure, dependencies
    - `external` — Focus on external research: docs, best practices, libraries, approaches
    - `both` — Explore both the codebase and external sources

## Agents

1. **cortex-team:scout** — Map what exists in the codebase related to this topic
2. **cortex-team:researcher** — Research external sources: documentation, best practices, alternatives

Collaboration style: **parallel then merge**. Scout and researcher work independently, findings are merged into a single report.

## Task

Investigate: "{{investigation-topic}}"

Focus: {{investigation-focus}}

**Project context**: (inject loaded context memory)

### Phase 1: Parallel Exploration

Launch **cortex-team:scout** and **cortex-team:researcher** in parallel (based on {{investigation-focus}}):

**Scout** (if focus is `codebase` or `both`):
- Find all code related to "{{investigation-topic}}"
- Map files, dependencies, patterns, and conventions
- Identify how this topic currently works (or where it would fit)
- Report with file:line references — facts only, no recommendations

**Researcher** (if focus is `external` or `both`):
- Find current documentation, tutorials, and best practices
- Identify common approaches and patterns used by the community
- Find relevant libraries, tools, or frameworks
- Report with sources — facts only, no recommendations

### Phase 2: Merge Findings

Combine findings from both agents into a coherent report. Identify:
- What we already have (codebase findings)
- What's available externally (research findings)
- Where the two overlap or conflict
- Knowledge gaps — what we still don't know

### Phase 3: Produce Investigation Report

Write the artifact to `.cortex/artifacts/{feature-id}.investigation.md` with this EXACT structure:

```markdown
---
artifact: investigation
feature: {{investigation-topic-slug}}
feature-id: {{feature-id}}
status: active
command: investigate
created: YYYY-MM-DD
source: null
agents: [scout, researcher]
---

# Investigation: {{investigation-topic}}

Focus: {{investigation-focus}}

## Codebase Findings

### Current State
(what exists today related to this topic)

### Relevant Files
| File | Relevance | Notes |
|---|---|---|
| `file:line` | (why it matters) | (key observation) |

### Patterns & Conventions
(how related things are currently done in this codebase)

### Dependencies
(internal and external dependencies related to this topic)

## External Research

### Best Practices
(what the community recommends, with sources)

### Common Approaches
| Approach | Pros | Cons | Source |
|---|---|---|---|
| (approach) | (advantages) | (disadvantages) | (URL or reference) |

### Relevant Libraries/Tools
| Library | Purpose | Maturity | Source |
|---|---|---|---|
| (name) | (what it does) | (stable/beta/experimental) | (URL) |

## Knowledge Gaps
(what we still don't know and would need to figure out)

## Raw Facts Summary
(bullet list of the most important facts discovered — no opinions, no recommendations)
```

Do NOT include recommendations or opinions. This is a fact-finding mission. Decisions come later.
