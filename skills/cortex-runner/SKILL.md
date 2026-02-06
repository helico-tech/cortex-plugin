---
name: cortex-runner
description: "Shared execution flow for all cortex-team commands. Handles parameter collection, memory loading, artifact management, agent execution, and memory storage. Do NOT invoke this skill directly — it is used internally by cortex-team commands."
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

## Step 3: Load Consumed Artifacts

Read the command's `## Artifacts` section. Check the `consumes:` declaration.

If `consumes: none`, skip this step.

Otherwise, for each artifact type listed:

1. The command specifies a `feature-id` parameter (e.g. `0001-auth-flow`). Use it to construct the filename: `.cortex/artifacts/{feature-id}.{artifact-type}.md`
2. Check if the file exists
3. If it exists, read it and make its content available for the task section
4. If it does NOT exist, **stop execution** and tell the user:
   > "This command requires a `{artifact-type}` artifact for feature `{feature-id}`.
   > Run the `{producing-command}` command first."

Summarize what artifacts were loaded.

## Step 4: Execute

Hand off to the calling command's task section. The command defines:
- Which agent(s) to launch and in what order
- What collaboration style to use (if multiple agents)
- The actual task prompt with `{{param}}` placeholders replaced, memory injected, and consumed artifacts injected

## Step 5: Produce Artifact

If the command declares `produces:` in its `## Artifacts` section, write the artifact file.

### Feature ID assignment

If this command CREATES a new feature (i.e. it has no `consumes:` and `produces: design`):

1. List all existing `.design.md` files in `.cortex/artifacts/`
2. Extract the highest 4-digit prefix (e.g. `0003` from `0003-notifications.design.md`)
3. Increment by 1 (→ `0004`)
4. Combine with the feature slug from the params: `0004-{feature-slug}`
5. This becomes the `feature-id` for all downstream commands

If this command is CONTINUING an existing feature (i.e. it `consumes:` a prior artifact), use the same `feature-id` from the consumed artifact's frontmatter.

### Artifact file format

Write to: `.cortex/artifacts/{feature-id}.{artifact-type}.md`

Every artifact MUST have this frontmatter:

```yaml
---
artifact: {artifact-type}
feature: {feature-slug}
feature-id: {NNNN-feature-slug}
status: active
command: {command-name}
created: YYYY-MM-DD
source: {consumed-artifact-filename or null}
agents: [{list of agents that participated}]
---
```

The body structure is defined by the command — each command specifies the exact template for its artifact.

### Numbering Convention

**This is mandatory.** All items in artifacts MUST use sequential numbered prefixes as defined by the command template. This is not optional formatting — it is a hard requirement for cross-referencing between artifacts.

Common numbering schemes used across commands:
- **REQ-001, REQ-002, ...** — Requirements (in design artifacts)
- **DEC-001, DEC-002, ...** — Decisions (in design artifacts)
- **TEST-001, TEST-002, ...** — Test cases (in design artifacts)
- **RISK-001, RISK-002, ...** — Risks (in design artifacts)
- **TASK-0001, TASK-0002, ...** — Tasks (in task artifacts, 4-digit)
- **FINDING-001, FINDING-002, ...** — Review findings (in review artifacts)
- **AUDIT-001, AUDIT-002, ...** — Audit findings (in audit artifacts)
- **FIX-001, FIX-002, ...** — Applied fixes (in tidy-report artifacts)
- **SKIP-001, SKIP-002, ...** — Skipped fixes (in tidy-report artifacts)

These IDs are how artifacts cross-reference each other. A task references `REQ-001` from the design. A test covers `REQ-002`. A review finding references `DEC-003`. Without numbered IDs, the traceability chain breaks.

**Rules:**
- Always start at 001 (or 0001 for TASK)
- Always sequential, no gaps
- Always use the exact prefix defined in the command template
- When appending items to an existing artifact (e.g. review adds tasks), continue from the last number

### Validation

After writing the artifact, verify:
1. The file exists at the expected path
2. The frontmatter contains all required fields (`artifact`, `feature`, `feature-id`, `status`, `command`, `created`, `agents`)
3. The body follows the structure defined in the command
4. **All items use the correct numbering prefixes** (REQ-NNN, TASK-NNNN, etc.)
5. **Numbering is sequential with no gaps**

If validation fails, fix the artifact before proceeding.

## Step 6: Write Journal

After execution completes, write a journal entry to `.cortex/memory/journal/`.

File path: `.cortex/memory/journal/YYYY-MM/YYYY-MM-DD-{feature-id}-{command-name}.md`

```markdown
---
date: YYYY-MM-DD
command: (command that was run)
feature-id: (feature-id)
agents: [(agents that participated)]
artifact-produced: (filename of artifact produced, or none)
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

## Step 7: Propose Context Updates

After writing the journal entry, review what was learned and decide if any findings are worth preserving as long-term project context.

If there are valuable findings, propose specific updates to `.cortex/memory/context/`:
- For each proposal, show the **file path** and the **exact content** you would write
- If the file already exists, show what you would add or change
- Keep proposals concise — context files should be scannable, not essays

Present all proposals to the user and ask for approval. Only write to `.cortex/memory/context/` if the user explicitly approves.

If nothing from this run is worth adding to long-term context, say so and skip this step. Not every run produces lasting knowledge — that's fine.
