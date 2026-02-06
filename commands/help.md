---
description: Get help with cortex-team — understand the system, find the right command for what you want to do, see examples
argument-hint: What you're trying to do (optional)
---

# Cortex Help

This command does NOT use the cortex-runner skill. It is a lightweight guide.

## Task

Help the user understand cortex-team and find the right command or workflow for what they want to do.

If $ARGUMENTS are provided, skip the project state check and jump straight to recommending commands based on the stated intent.

### Step 1: Project State Check

Quickly scan the project to understand where the user is:

1. **Check if cortex is initialized**: Does `.cortex/` exist?
   - If not: tell the user to run `/cortex-team:cortex-init` first, explain what it does, and stop.

2. **Check for in-progress work**: Scan `.cortex/artifacts/` for active artifacts.
   - List any active features with their current state (which artifacts exist = how far along the flow they are)
   - Check for active workflow-state artifacts (workflows in progress)
   - Present a brief summary: "You have 2 features in progress: 0001-auth-flow (designed, planned, implementing), 0003-notifications (designed)"

3. **Check memory**: Does `.cortex/memory/context/` have any files?
   - If empty: mention that the team hasn't built up context memory yet — it'll grow as commands run

Present this as a quick status, not a wall of text. 2-3 lines max.

### Step 2: Understand Intent

Ask the user what they're trying to do. Use AskUserQuestion with these options:

**"What do you want to do?"**

- `Start something new` — I have an idea or feature to build
- `Continue existing work` — I have in-progress work to pick up
- `Fix a problem` — There's a bug or issue to address
- `Improve code quality` — Clean up, refactor, or audit existing code
- `Explore or research` — Investigate something before committing to a direction
- `Understand the system` — Explain how cortex-team works

### Step 3: Recommend Based on Intent

Based on the user's answer, recommend the right command or workflow:

#### Start something new

"Starting from scratch? Here's the path:"

1. **If the idea is vague:** Start with brainstorming
   ```
   /cortex-team:brainstorm <your idea>
   ```

2. **If the idea is clear:** Jump straight to design
   ```
   /cortex-team:design <feature-name>
   ```

3. **If you want the full guided flow:** Use the feature workflow
   ```
   /cortex-team:workflow feature
   ```

Explain the flow briefly: brainstorm (optional) → design → plan → implement → review → validate

#### Continue existing work

List the in-progress features found in step 1. For each, show where it is and what to run next:

- If has design but no tasks: `/cortex-team:plan {feature-id}`
- If has tasks with pending items: `/cortex-team:implement {feature-id}`
- If all tasks done, no review: `/cortex-team:review {feature-id}`
- If review passed, no validation: `/cortex-team:validate {feature-id}`
- If review failed: `/cortex-team:implement {feature-id}` (review findings are already in the task list)

If there's an active workflow, suggest resuming it:
```
/cortex-team:workflow <workflow-name>
```

#### Fix a problem

```
/cortex-team:fix <describe the bug>
```

Explain: fix triages automatically. Small bugs get fixed directly. Larger issues get routed to design.

For urgent fixes, suggest the hotfix workflow:
```
/cortex-team:workflow hotfix
```

#### Improve code quality

Present the options:

- **Quick cleanup:** `/cortex-team:tidy <area>` — find and fix code quality issues
- **Health check:** `/cortex-team:audit <area>` — read-only assessment, no changes
- **Structural change:** `/cortex-team:refactor <area>` — investigate and design a refactor
- **Full gardening day:** `/cortex-team:workflow maintenance` — audit → tidy → curate

#### Explore or research

Present the options:

- **Codebase + web research:** `/cortex-team:investigate <topic>` — facts only, no decisions
- **Open exploration:** `/cortex-team:brainstorm <idea>` — explore until something crystallizes
- **Research then design:** `/cortex-team:workflow spike` — investigate → design, stop before building

#### Understand the system

Provide a concise explanation of cortex-team:

**The system in 30 seconds:**
- cortex-team standardizes how your team uses Claude across projects
- **Commands** are what you run: `/cortex-team:design`, `/cortex-team:implement`, etc.
- **Agents** are specialized roles (scout, architect, pragmatist, implementer, reviewer, tester, researcher, writer) that work in teams — never solo
- **Artifacts** are standardized outputs (design docs, task lists, reviews) that chain between commands. Stored in `.cortex/artifacts/`
- **Memory** accumulates project knowledge in `.cortex/memory/context/` (curated, always loaded) and `.cortex/memory/journal/` (raw history)
- **Workflows** orchestrate multiple commands: `/cortex-team:workflow feature`

**The core flow:**
```
brainstorm → design → plan → implement → review ↔ implement → validate
```

**All commands:**

| Command | What it does |
|---|---|
| `cortex-init` | Initialize `.cortex/` in a project |
| `brainstorm` | Open exploration until an idea crystallizes |
| `design` | Produce a design document with requirements, decisions, tests |
| `plan` | Break a design into implementable tasks |
| `implement` | Execute tasks with scout→implement→test loop |
| `review` | Adversarial code review with pass/fail verdict |
| `validate` | Verify implementation meets design requirements |
| `fix` | Triage and fix a bug (or route to design) |
| `refactor` | Investigate and design a refactor |
| `tidy` | Find-fix-verify code quality issues |
| `audit` | Read-only codebase health report |
| `investigate` | Fact-finding with scout + researcher |
| `curate` | Process journals into context memory |
| `workflow` | Run a multi-step workflow |
| `workflow-designer` | Create a project-specific workflow |
| `command-designer` | Create a project-specific command |
| `help` | You are here |

**Available workflows:** `feature`, `hotfix`, `refactor`, `maintenance`, `spike`
