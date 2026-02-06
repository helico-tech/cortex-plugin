---
description: Initialize a project for cortex-team by creating the .cortex/ folder structure
---

# Cortex Init

Initialize the `.cortex/` directory structure in the current project.

## Steps

1. Check if `.cortex/` already exists. If it does, tell the user and stop.

2. Create the following directory structure:

```
.cortex/
  memory/
    conventions/
    domain-model/
    decisions/
    lessons-learned/
    reflections/
```

3. Create a `.gitkeep` file in each empty memory subdirectory so they are tracked by git.

4. Tell the user the initialization is complete and show the created structure.
