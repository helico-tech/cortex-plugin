---
description: Document why non-obvious code exists — write a code marker and a context memory entry so the "why" is never lost
argument-hint: What's the non-obvious thing? (file, function, or describe it)
---

# Rationale

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: none (writes to code and context memory instead)
- consumes: none

## Params

- **what**:
  - type: string
  - question: "What's the non-obvious code? (file:line, function name, or describe the weird thing)"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **why**:
  - type: string
  - question: "Why does it exist? What breaks if someone removes or changes it?"
  - required: true

## Agents

1. **cortex-team:scout** — Find the exact code location and understand the surrounding context
2. **cortex-team:writer** — Write the rationale entry clearly and concisely

Collaboration style: **sequential**. Scout finds the code, writer documents it.

## Task

Document a rationale for: "{{what}}"

**Project context**: (inject loaded context memory)

### Phase 1: Find the Code

Launch the **cortex-team:scout**:
- Locate the exact code being documented
- Identify the file and line number
- Understand the surrounding code context
- Report: file path, line number, the code snippet, and what it does

### Phase 2: Write the Rationale

Launch the **cortex-team:writer** with the scout's findings and the user's explanation ("{{why}}"):

Determine the next RAT-NNN number:
1. Read `.cortex/memory/context/rationales.md` if it exists
2. Find the highest RAT-NNN number
3. Increment by 1 (or start at RAT-001 if no file exists)

Write TWO things:

**1. Code marker** — Add a comment at the code location:

```
// @rationale RAT-NNN: (short title)
```

Place the marker on the line ABOVE the non-obvious code. Use the file's existing comment syntax (`//`, `#`, `/* */`, `--`, etc.).

The marker is intentionally short — one line. The full story is in memory.

**2. Memory entry** — Append to `.cortex/memory/context/rationales.md`:

```markdown
## RAT-NNN: (short title)
- **Location**: `file:line`
- **Code**: `(the non-obvious code snippet, 1-3 lines)`
- **Why**: (clear explanation of why this code exists and what it prevents)
- **What breaks**: (specifically what goes wrong if this code is removed or changed)
- **Added**: YYYY-MM-DD
- **Source**: (feature-id or "manual" if added outside a feature flow)
- **DO NOT**: (explicit warning — what NOT to do with this code)
```

If `rationales.md` doesn't exist yet, create it with this header:

```markdown
# Rationales

Non-obvious code decisions that need preserved context. Every entry here is linked to a `@rationale RAT-NNN` marker in the codebase.

These are loaded into every cortex command. Agents: respect these rationales. Do NOT modify, remove, or "improve" code marked with @rationale without understanding and explicitly acknowledging the rationale.
```

### Phase 3: Confirm

Show the user:
1. The code marker that was added (with surrounding context)
2. The memory entry that was written
3. The RAT-NNN ID for future reference

### Skip Cortex-Runner Post-Steps

This command writes directly to code and context memory. Skip steps 5-7 of the cortex-runner (no artifact, no journal, no context proposal — we already wrote to context).
