---
description: Capture a thought into project memory — quick deposit into the journal for curate to promote later
argument-hint: What do you want to remember? (fact, insight, convention, correction — anything)
---

# Memoize

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: none (writes to journal instead)
- consumes: none

## Params

- **thought**:
  - type: string
  - question: "What do you want to remember? (state it however you want — it will be sharpened)"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.

## Agents

1. **cortex-team:writer** — Scan memory for conflicts, sharpen the thought, write the journal entry

Collaboration style: **solo**. Single agent handles the full flow.

## Task

Memoize: "{{thought}}"

**Project context**: (inject loaded context memory)

### Phase 1: Scan, Sharpen, Write (Writer)

Launch the **cortex-team:writer** with the user's thought and all loaded context memory.

**Instructions:**

You are writing a memo journal entry. The user had a thought they want preserved in project memory. Your job is to check it against what's already known, sharpen it, and write a clean journal entry.

**Step 1: Scan loaded memory for conflicts**

Read through the context memory that was loaded in step 2. Do a quick check for:

1. **Contradictions** — existing context that says the OPPOSITE of the user's thought
2. **Overlaps** — existing context that says the SAME THING (or close enough to be redundant)

This is a lightweight keyword-level scan, not deep semantic analysis. If you spot something obvious, note it. If nothing jumps out, move on. Do not spend time on exhaustive searching.

**Step 2: Sharpen the thought**

Take the raw thought and:
- Fix grammar and clarity (do not change meaning)
- Make it specific — replace vague references with concrete ones where possible
- Make it scannable — someone should understand the core point in 5 seconds
- Keep it SHORT: 3-8 lines. This is a note, not an essay.

Do NOT editorialize. Do NOT add analysis the user did not provide. Do NOT expand the scope. Light sharpening means cleaning up, not rewriting.

**Step 3: Generate the slug**

Create a filename slug from the sharpened thought:
- 2-4 words, lowercase, hyphenated
- Derived from the core concept, not the full sentence
- Examples: `postgres-pool-limit`, `zod-request-validation`, `auth-no-token-cache`
- Only lowercase letters, numbers, and hyphens

If a file already exists at the target path (same slug on the same day), append `-2`, `-3`, etc. to disambiguate.

**Step 4: Write the journal entry**

Write to: `.cortex/memory/journal/YYYY-MM/YYYY-MM-DD-memo-{slug}.md`

Create the `YYYY-MM/` directory if it does not exist.

Use this EXACT format:

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

If you found contradictions or overlaps in step 1, append:

```markdown
### Note

Potentially conflicts with (file path and section). Consider reviewing during curate.
```

**Step 5: Confirm**

Show the user:
1. The journal file path that was written
2. The sharpened memo content
3. Any contradiction/overlap notes (if applicable)
4. Suggest: "Run `/cortex-team:curate` when you have a batch of memos to promote to project context."

### Skip Cortex-Runner Post-Steps

This command writes directly to the journal. Skip steps 3, 5, and 7 of the cortex-runner:
- Step 3: no consumed artifacts
- Step 5: no produced artifact — the journal entry IS the output
- Step 7: no context proposal — curate handles promotion to context later

Step 6 (write journal): the writer already wrote the journal entry in step 4. Verify the file exists and has valid frontmatter, but do not write a second entry.
