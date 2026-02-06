---
description: Assess how well this codebase supports agentic development — evaluate feedback loops, runnability, introspectability, and navigability
argument-hint: Area to assess (directory, module, or 'everything')
---

# Assess

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: assessment
- consumes: none

## Params

- **assess-scope**:
  - type: string
  - question: "What area should be assessed? (file path, directory, module name, or 'everything')"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **assess-depth**:
  - type: choice
  - question: "How deep should the assessment go?"
  - options:
    - `static` — Read-only analysis: scan files, config, structure, test presence
    - `active` — Active probing: run tests, try to start the app, trace execution paths
    - `full` — Both static analysis and active probing

## Agents

1. **scout** — Map the codebase from an agent's perspective: structure, entry points, test infra, docs, tooling
2. **architect** — Evaluate introspectability and navigability: types, boundaries, naming, dependency clarity
3. **tester** — Evaluate feedback loops and runnability: test levels, coverage, scripts, dev server, CI
4. **pragmatist** — Review all findings and proposed improvements, ruthlessly prioritize by agent impact

Collaboration style: **sequential + parallel**. Scout maps first. Then architect and tester assess in parallel (each from their own angle). Pragmatist reviews everything last and culls low-impact recommendations.

## Task

Assess agentic readiness of "{{assess-scope}}" at depth: {{assess-depth}}

**Project context**: (inject loaded context memory)

### Phase 1: Mapping

Launch the **scout**:
- File structure: organization, naming conventions, consistency across the scope
- Entry points: scripts in package.json, Makefile, docker-compose, CLI commands
- Test infrastructure: test files, test config (jest/vitest/pytest/etc.), test scripts, CI config
- Documentation: CLAUDE.md, README, API docs, inline types, JSDoc/TSDoc
- Dev tooling: linters, formatters, type checking, dev servers, watch modes
- Environment: .env.example, setup scripts, seed data, fixtures, docker setup
- Report raw facts with file:line references — no opinions

### Phase 2: Parallel Assessment

Launch **architect** and **tester** in parallel:

**Architect** (introspectability + navigability):
- Can you find things by grepping? Consistent naming, no magic strings, no dynamic dispatch hiding call sites
- Are module boundaries clear? Can you reason about blast radius of a change?
- Are types present and useful? Do they tell you what flows where?
- Is the dependency graph understandable? Circular dependencies? Hidden coupling?
- Is file organization consistent? Can you predict where something lives?
- Is there a CLAUDE.md or equivalent that orients an agent?
- If `active` depth: try to trace a representative request or operation end-to-end through the codebase — report how easy or hard it was to follow

**Tester** (feedback + runnability):
- What test levels exist? Unit, integration, e2e? What's the ratio?
- Are there obvious gaps in test coverage at any level?
- Can tests be run with a single command? What command? How long do they take?
- Is there a dev server or startup script? What command starts it?
- Are there seed data, fixtures, or test databases?
- Is there CI configuration? What does it run?
- If `active` depth: actually run the test suite — report pass/fail count, duration, and quality of error messages on failure. Try to start the application — report what happens.

### Phase 3: Pragmatist Review

Give the **pragmatist** all findings from scout, architect, and tester, plus the draft improvement tasks.

The pragmatist's job:
- Kill any recommendation that's theoretical, aspirational, or low-impact
- Kill anything that's general code quality rather than specifically agentic readiness
- Order remaining improvements by bang-for-buck: smallest effort × highest agent impact first
- Challenge vague recommendations — every task must be concrete enough that someone could act on it today

### Phase 4: Produce Assessment Report

Write the artifact to `.cortex/artifacts/{feature-id}.assessment.md` with this EXACT structure:

```markdown
---
artifact: assessment
feature: {{assess-scope-slug}}
feature-id: {{feature-id}}
status: active
command: assess
created: YYYY-MM-DD
source: null
agents: [scout, architect, tester, pragmatist]
---

# Assessment: {{assess-scope}}

Depth: {{assess-depth}}

## Scorecard

| Dimension | Rating | Summary |
|---|---|---|
| Feedback | strong/adequate/weak/missing | (one-line) |
| Runnability | strong/adequate/weak/missing | (one-line) |
| Introspectability | strong/adequate/weak/missing | (one-line) |
| Navigability | strong/adequate/weak/missing | (one-line) |

Overall agentic readiness: **strong/adequate/weak/missing**

## Feedback

### What Exists
(test levels found, test config, CI setup, pass rate if active depth)

### What's Missing
(gaps in test coverage, missing test levels, broken or flaky tests)

### Agent Impact
(how this affects an agent's ability to verify its own work)

## Runnability

### What Exists
(dev scripts, docker, startup commands, seed data, fixtures)

### What's Missing
(manual steps required, undocumented dependencies, missing fixtures)

### Agent Impact
(how this affects an agent's ability to run and validate changes)

## Introspectability

### What Exists
(types, API contracts, module boundaries, naming patterns)

### What's Missing
(untyped areas, magic strings, unclear boundaries, implicit behavior)

### Agent Impact
(how this affects an agent's ability to understand the code)

## Navigability

### What Exists
(CLAUDE.md, file organization, naming conventions, grep-friendliness)

### What's Missing
(inconsistent naming, deep nesting, undocumented structure, magic dispatch)

### Agent Impact
(how this affects an agent's ability to find things)

## Improvement Tasks

### High Impact

#### IMPROVE-001: (title)
- **Dimension**: feedback/runnability/introspectability/navigability
- **What**: (concrete action — specific enough to act on today)
- **Why**: (what it unlocks for agentic development)
- **Effort**: small/medium/large

#### IMPROVE-002: ...

### Medium Impact

#### IMPROVE-003: ...

### Low Impact / Nice-to-Have

#### IMPROVE-004: ...
```

Ratings guide:
- **strong**: Agent can operate effectively with minimal friction
- **adequate**: Agent can work but will hit friction points regularly
- **weak**: Agent will struggle significantly, frequent human intervention needed
- **missing**: This dimension is essentially absent, agent is flying blind
