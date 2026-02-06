---
name: cortex-runner
description: "Shared execution flow for all cortex-team commands. Handles parameter collection, memory loading, agent execution, and memory storage. Do NOT invoke this skill directly — it is used internally by cortex-team commands."
---

# Cortex Runner

This skill defines the standard execution flow for all cortex-team prompt templates. Every cortex command follows these steps in order.

## Step 1: Collect Parameters

The calling command defines a `params` section. For each parameter:

- If `type: string` — ask the question, accept freeform text
- If `type: choice` — use AskUserQuestion with the defined options
- If `type: boolean` — ask as a yes/no question
- If a `default` is provided and the user gives no answer, use the default
- If `required: true` (the default), do not proceed without an answer

Ask one question at a time. Wait for the answer before asking the next.

Store all answers — they will replace `{{param-name}}` placeholders in the command's task section.

## Step 2: Load Memory

Read from the project's `.cortex/memory/` directory. For any path that doesn't exist, skip silently.

**Always load:**
- All `.md` files in `.cortex/memory/conventions/`
- All `.md` files in `.cortex/memory/domain-model/`
- All `.md` files in `.cortex/memory/lessons-learned/`

**Feature-scoped** (only if a `feature-name` param was collected):
- `.cortex/memory/decisions/{{feature-name}}.md`
- All `.md` files in `.cortex/memory/reflections/{{feature-name}}/`

After loading, briefly summarize what memory was found so the user knows what context is available.

## Step 3: Execute

Hand off to the calling command's task section. The command defines:
- Which agent(s) to launch
- What collaboration style to use (if multiple agents)
- The actual task prompt with `{{param}}` placeholders replaced and memory injected

## Step 4: Store Memory

After execution completes, write results to `.cortex/memory/`. Only write if there's something meaningful to store.

**What to store:**
- Key decisions → `.cortex/memory/decisions/{{feature-name}}.md` (append if exists)
- Agent reflections → `.cortex/memory/reflections/{{feature-name}}/{agent-name}.md`
- Lessons learned → `.cortex/memory/lessons-learned/{topic}.md` (append if exists)

**Memory file format** — all memory files use markdown with structured frontmatter:

```markdown
---
date: YYYY-MM-DD
feature: (feature name if applicable)
agent: (agent that produced this)
type: decision|reflection|lesson
---

(content here)
```
