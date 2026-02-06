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

Read all `.md` files from `.cortex/memory/context/`. Skip silently if the directory doesn't exist or is empty.

This is the curated project knowledge — conventions, domain model, key decisions. It is always loaded in full.

After loading, briefly summarize what memory was found so the user knows what context is available.

## Step 3: Execute

Hand off to the calling command's task section. The command defines:
- Which agent(s) to launch
- What collaboration style to use (if multiple agents)
- The actual task prompt with `{{param}}` placeholders replaced and memory injected

## Step 4: Store Memory

After execution completes, write a journal entry to `.cortex/memory/journal/`.

**Journal entry format:**

File path: `.cortex/memory/journal/YYYY-MM/YYYY-MM-DD-{topic}-{command-name}.md`

```markdown
---
date: YYYY-MM-DD
command: (command that was run)
topic: (feature or area worked on)
agents: [(agents that participated)]
---

## Summary
(one-sentence summary of what happened)

## Context
(what problem was being solved)

## Key Findings
(bullet points of important observations)

## Decisions Made
(what was decided, with brief rationale)

## Open Questions
(anything unresolved that needs follow-up)
```

Only write a journal entry if there's something meaningful to record. Don't create empty or boilerplate entries.

**Important:** Do NOT write directly to `.cortex/memory/context/`. Context files are curated — they are maintained by humans or through an explicit curation process, never auto-written by command execution.
