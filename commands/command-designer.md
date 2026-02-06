---
description: Design a project-specific command collaboratively — create a command that follows cortex conventions with proper agents, params, artifacts, and phased tasks
argument-hint: Command name or purpose
---

# Command Designer

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: none (writes to `.claude/commands/` instead)
- consumes: none

## Params

- **command-purpose**:
  - type: string
  - question: "What should this command do? Describe the goal or give it a name."
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.

## Agents

1. **cortex-team:architect** — Propose command structure: params, agents, phases, artifact contract
2. **cortex-team:pragmatist** — Challenge unnecessary complexity, push for the minimal command that achieves the goal

Collaboration style: **iterative with user**. Architect proposes, pragmatist challenges, user decides. One decision at a time.

## Task

Design a project command for: "{{command-purpose}}"

**Project context**: (inject loaded context memory)

### Phase 1: Understand the Goal

Ask the user one question at a time to understand:
- What does this command do?
- Is it a one-shot task or part of a larger flow?
- Does it need to consume artifacts from prior commands?
- Does it produce something that other commands should consume?

Do NOT dump all questions at once. One at a time.

### Phase 2: Define the Artifact Contract

Help the user decide:

**Consumes:** Does this command need input from prior commands?
- Show available artifact types: design, tasks, review, validation, tidy-report, audit, investigation, curation
- If yes, which artifacts? This means the command requires a feature-id parameter.
- If none, this command starts fresh.

**Produces:** Does this command generate a standardized output?
- If yes, define the artifact type name and structure
- If this is a new artifact type, design its template with the user
- If no, the command just does its work without producing a tracked artifact

### Phase 3: Pick Agents

Present the available agent roster:

| Agent | Strength | Constraint |
|---|---|---|
| scout | Maps codebase, reports facts | Does NOT suggest changes |
| architect | Structural decisions, design | Does NOT write code |
| pragmatist | Challenges complexity, YAGNI | Tears down, does NOT build |
| implementer | Writes code, follows patterns | Does NOT question design |
| reviewer | Adversarial code review | Does NOT write code |
| tester | Test strategy + writing | Tests only, no production code |
| researcher | Web research, docs, APIs | Reports findings, no opinions |
| writer | Developer documentation | Docs only, no code evaluation |

Help the user select which agents this command needs. Every command should use at least 2 agents (teams, not solo). Define the collaboration style:
- **sequential** — agents work in order, each builds on the previous
- **parallel then merge** — agents work independently, findings merged
- **sequential with debate** — one proposes, another challenges
- **iterative loop** — repeat until done (scout→implement→test→repeat)

### Phase 4: Define Params

Help the user define command parameters:
- What inputs does the command need?
- For each param: name, type (string/choice/boolean), question, required, default
- Which params come from $ARGUMENTS?

### Phase 5: Design the Task

Launch the **cortex-team:architect** to propose the phased task structure:
- Break the command's work into phases
- Assign agents to each phase
- Define what each agent does in each phase
- Ensure the phases flow logically

Launch the **cortex-team:pragmatist** to challenge:
- Are all phases necessary?
- Is any agent doing redundant work?
- Could this be simpler?

Iterate with the user until they're happy with the task structure.

### Phase 6: Validate

Before saving, verify:
- **At least 2 agents** are assigned
- **Artifact contract is valid**: if it consumes artifacts, those artifact types exist in the cortex ecosystem
- **Params are complete**: every {{placeholder}} in the task has a corresponding param
- **Follows cortex-runner flow**: the command says "Use the **cortex-runner** skill to execute this template."
- **Artifact template** (if producing): includes all required frontmatter fields and uses proper numbering prefixes

### Phase 7: Save

Write the command to `.claude/commands/{command-name}.md` using this format:

```markdown
---
description: (what this command does)
argument-hint: (hint for arguments, if applicable)
---

# (Command Title)

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: (artifact-type or none)
- consumes: (artifact-type(s) or none)

## Params

(param definitions)

## Agents

(numbered agent list with collaboration style)

## Task

(phased task with {{param}} placeholders)
```

After saving, tell the user:
> "Command '{command-name}' saved to `.claude/commands/{command-name}.md`.
> Invoke it with: `/{command-name}`
> It follows the cortex-runner flow and will use cortex-team agents."

### Skip the Cortex-Runner Post-Steps

This command does NOT produce a standard artifact, so skip steps 5-7 of the cortex-runner (produce artifact, journal, propose context). The command file IS the output, written directly to `.claude/commands/`.
