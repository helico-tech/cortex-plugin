---
description: Review the architecture of a feature area using the architect agent
argument-hint: Optional feature area to review
---

# Architecture Review

You are running a cortex-team prompt template. Follow these steps exactly.

## Step 1: Collect Parameters

Ask the user the following questions one at a time. Wait for each answer before asking the next.

- **feature-name** (required): "What feature or area should I review?" (Use $ARGUMENTS if provided, otherwise ask)
- **user-involvement** (required, choose one): "How involved do you want to be?"
  - `hands-off` — Work autonomously, only interrupt for blockers
  - `during-unknowns` — Check in when you hit ambiguity or need a decision
  - `every-step` — Walk me through each finding before moving on

## Step 2: Load Memory

Read the following from the project's `.cortex/memory/` directory. Skip any that don't exist — don't error, just note what was available.

- **Always load**: All files in `.cortex/memory/conventions/`
- **Always load**: All files in `.cortex/memory/domain-model/`
- **Feature-scoped**: `.cortex/memory/decisions/{{feature-name}}.md`
- **Feature-scoped**: All files in `.cortex/memory/reflections/{{feature-name}}/`
- **Always load**: All files in `.cortex/memory/lessons-learned/`

Summarize what memory was loaded so the user knows what context you're working with.

## Step 3: Execute with Architect Agent

Launch the **architect** agent with the following task:

> Review the architecture of the "{{feature-name}}" area in this codebase.
>
> **Team conventions**: (inject loaded conventions)
> **Domain model**: (inject loaded domain model)
> **Prior decisions**: (inject loaded decisions for this feature)
> **Lessons learned**: (inject loaded lessons)
>
> Analyze the current architecture and produce:
> 1. Current architecture summary — what exists today
> 2. Strengths — what's working well
> 3. Concerns — structural issues, violations of conventions, complexity hotspots
> 4. Recommendations — specific, actionable improvements

Respect the user's **user-involvement** preference throughout.

## Step 4: Store Memory

After the architect agent completes, store the results:

- Write key architectural decisions to `.cortex/memory/decisions/{{feature-name}}.md` (append if exists)
- Write the architect's self-reflection to `.cortex/memory/reflections/{{feature-name}}/architect.md`
- If any new lessons were learned, append to a relevant file in `.cortex/memory/lessons-learned/`

Use structured markdown with frontmatter for all memory files:

```markdown
---
date: YYYY-MM-DD
feature: {{feature-name}}
agent: architect
type: decision|reflection|lesson
---

(content here)
```
