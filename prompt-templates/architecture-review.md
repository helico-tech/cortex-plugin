---
description: Review the architecture of a feature area
params:
  feature-name:
    type: string
    question: "What feature or area should I review?"
    required: true
  user-involvement:
    type: choice
    question: "How involved do you want to be?"
    options:
      hands-off: "Work autonomously, only interrupt for blockers"
      during-unknowns: "Check in when you hit ambiguity or need a decision"
      every-step: "Walk me through each finding before moving on"
agents:
  - name: architect
    role: lead
---

# Architecture Review: {{feature-name}}

Review the architecture of the "{{feature-name}}" area in this codebase.

## Context

**Team conventions**: (inject loaded conventions)
**Domain model**: (inject loaded domain model)
**Prior decisions**: (inject loaded decisions for this feature)
**Lessons learned**: (inject loaded lessons)

## Task

Analyze the current architecture and produce:

1. **Current architecture summary** — what exists today
2. **Strengths** — what's working well
3. **Concerns** — structural issues, violations of conventions, complexity hotspots
4. **Recommendations** — specific, actionable improvements

Respect the user's **{{user-involvement}}** involvement preference throughout.
