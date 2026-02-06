---
description: Design a feature — produce a comprehensive design document with requirements, decisions, implementation approach, test strategy, and risks
argument-hint: Feature description
---

# Design

Use the **cortex-runner** skill to execute this template.

## Artifacts

- produces: design
- consumes: none

## Params

- **feature-name**:
  - type: string
  - question: "What feature are you designing? (short name for the slug, e.g. 'auth-flow', 'notification-system')"
  - required: true
  - Use $ARGUMENTS if provided, otherwise ask.
- **feature-description**:
  - type: string
  - question: "Describe the feature. What problem does it solve? What should it do?"
  - required: true
- **user-involvement**:
  - type: choice
  - question: "How involved do you want to be during the design process?"
  - options:
    - `hands-off` — Work autonomously, only interrupt for blockers
    - `during-unknowns` — Check in when you hit ambiguity or need a decision
    - `every-step` — Walk me through each section before moving on

## Agents

1. **researcher** — Research current best practices, library docs, and patterns relevant to this feature
2. **scout** — Explore the existing codebase to understand current patterns, conventions, and related code
3. **architect** — Design the solution: structure, components, data flow, decisions with trade-offs
4. **pragmatist** — Challenge the architect's design: is anything over-engineered? What's the simpler path?
5. **tester** — Define test strategy: what to test, how, risk areas, edge cases, pass/fail criteria

Collaboration style: **sequential with debate**. Researcher and scout gather context. Architect proposes. Pragmatist challenges. Architect revises. Tester defines test strategy. Final design incorporates all perspectives.

## Task

Design the feature "{{feature-name}}": {{feature-description}}

**Project context**: (inject loaded context memory)

### Phase 1: Research & Discovery

Launch the **researcher** and **scout** agents in parallel:

- **Researcher**: Find current best practices, library documentation, and patterns relevant to "{{feature-name}}". Report findings with sources.
- **Scout**: Explore the existing codebase. Find related code, existing patterns, conventions, and integration points relevant to "{{feature-name}}". Report with file:line references.

### Phase 2: Architecture & Challenge

Launch the **architect** with the research and scout findings:

- Propose the design: components, data flow, integration points, technology choices
- Make decisions with explicit trade-offs and rejected alternatives
- Be specific: file paths, function signatures, data structures

Then launch the **pragmatist** to challenge the architect's proposal:

- What's over-engineered? What's the simpler alternative?
- What's being built for hypothetical future requirements?
- What's the delivery cost vs. value?

The architect revises based on the pragmatist's valid challenges. If they disagree on something fundamental, present both positions to the user for a decision.

### Phase 3: Test Strategy

Launch the **tester** with the revised design:

- Define what to test and at what level (unit, integration, e2e)
- Identify risk areas and edge cases
- Define explicit pass/fail criteria for validation
- Note attention points for the implementation team

### Phase 4: Produce Design Document

Write the artifact to `.cortex/artifacts/{{feature-id}}.design.md` with this EXACT structure:

```markdown
---
artifact: design
feature: {{feature-name}}
feature-id: {{feature-id}}
status: active
command: design
created: YYYY-MM-DD
source: null
agents: [researcher, scout, architect, pragmatist, tester]
---

# Design: {{feature-name}}

## Problem Statement
(what problem this feature solves and why it matters)

## Requirements
- REQ-001: (requirement description) | priority: must/should/could
- REQ-002: ...
(each requirement is numbered and prioritized)

## Decisions
### DEC-001: (decision title)
- **Decided**: (what was chosen)
- **Rationale**: (why)
- **Alternatives rejected**: (what else was considered and why not)

### DEC-002: ...

## Implementation Approach
### Components
(component breakdown with responsibilities and interfaces)

### Data Flow
(how data moves through the system)

### Files to Create/Modify
(specific file paths with descriptions of changes)

### Integration Points
(where this connects to existing code, with file:line references)

## Test Strategy
### Risk Areas
(what can go wrong, ordered by severity)

### Test Plan
- TEST-001: (what to test) | level: unit/integration/e2e | covers: REQ-NNN
- TEST-002: ...
(each test is numbered, leveled, and linked to a requirement)

### Edge Cases
(specific edge cases to cover)

### Pass/Fail Criteria
(explicit, checkable conditions for validation)

## Risks & Attention Points
- RISK-001: (risk description) | impact: high/medium/low | mitigation: (how to handle)
- RISK-002: ...

## Open Questions
(anything unresolved that needs human input)
```

Respect the user's **{{user-involvement}}** preference throughout.
