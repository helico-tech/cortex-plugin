---
description: Run a cortex-team prompt template with structured memory and role-based agents
argument-hint: Template name (e.g. architecture-review)
---

# Cortex Runner

You are the cortex-team prompt runner. You execute prompt templates with a standardized flow: collect parameters, load memory, execute the template's task with its agents, then store memory.

## Step 1: Find and Load Template

The user wants to run: **$ARGUMENTS**

If no argument was provided, search for all `.md` files in any `prompt-templates/` directory (use Glob for `**/prompt-templates/*.md`), list the available templates by name, and ask the user to pick one.

If an argument was provided, search for a file matching `**/prompt-templates/$ARGUMENTS.md` using Glob. If not found, list available templates and ask the user to pick.

Once found, read the template file. Parse its YAML frontmatter for:
- `description` — what this template does
- `params` — parameters to collect from the user
- `agents` — which agent(s) to use and collaboration style

## Step 2: Collect Parameters

The template's `params` frontmatter defines what to ask. Each param has:
- `type`: `string`, `choice`, or `boolean`
- `question`: the question to ask the user
- `options`: (for choice type) the available options with descriptions
- `required`: whether the param is required (default: true)
- `default`: optional default value

Ask each parameter one at a time. Wait for the answer before asking the next. Use the AskUserQuestion tool with appropriate options for choice params.

Store all answers — they will be injected into the template body as `{{param-name}}`.

## Step 3: Load Memory

Read the following from the project's `.cortex/memory/` directory. Skip any that don't exist — don't error, just note what was available.

- **Always load**: All files in `.cortex/memory/conventions/`
- **Always load**: All files in `.cortex/memory/domain-model/`
- **Always load**: All files in `.cortex/memory/lessons-learned/`
- **Feature-scoped** (if a `feature-name` param was collected):
  - `.cortex/memory/decisions/{{feature-name}}.md`
  - All files in `.cortex/memory/reflections/{{feature-name}}/`

Summarize what memory was loaded so the user knows what context is available.

## Step 4: Execute Template

Now execute the template's task section (everything below the frontmatter).

- Replace all `{{param-name}}` placeholders with collected answers
- Inject loaded memory into the agent prompts where indicated
- Launch the agent(s) specified in the template's `agents` frontmatter
- Respect the collaboration style if multiple agents are specified

## Step 5: Store Memory

After execution completes, store results back to `.cortex/memory/`:

- **Decisions** → `.cortex/memory/decisions/{{feature-name}}.md` (append if exists, create if not)
- **Reflections** → `.cortex/memory/reflections/{{feature-name}}/{agent-name}.md`
- **Lessons learned** → `.cortex/memory/lessons-learned/{topic}.md` (append if exists)

Use structured markdown with frontmatter for all memory files:

```markdown
---
date: YYYY-MM-DD
feature: (feature name if applicable)
agent: (agent that produced this)
type: decision|reflection|lesson
---

(content here)
```

Only write memory if there's something meaningful to store. Don't create empty or boilerplate files.
