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
  config.json
  memory/
    conventions/
    domain-model/
    decisions/
    lessons-learned/
    reflections/
```

3. Create `.cortex/config.json` with this initial content:

```json
{
  "project": {
    "name": "",
    "description": "",
    "techStack": []
  }
}
```

4. Ask the user to fill in the config values:
   - Project name
   - Short project description
   - Tech stack (comma-separated list)

5. Write the answers to `.cortex/config.json`.

6. Create a `.cortex/memory/conventions/.gitkeep` file, and do the same for all other empty memory subdirectories, so they are tracked by git.

7. Tell the user the initialization is complete and show the created structure.
