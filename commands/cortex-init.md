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
    context/
    journal/
  artifacts/
  workflows/
```

3. Create a `.gitkeep` file in each empty subdirectory so they are tracked by git.

4. **Check MCP server availability.** cortex-team works best with these MCP servers installed:

   | MCP Server | Used by | What it enables |
   |---|---|---|
   | **context7** | researcher | Fast, reliable library/framework documentation lookups |
   | **playwright** | tester, reviewer | Browser-based testing and visual validation |

   For each, check if its tools are available (e.g. try to detect `mcp__context7__resolve-library-id` or `mcp__playwight__browser_snapshot` in your available tools).

   - If **all are available**: "All recommended MCP servers detected."
   - If **some are missing**: list the missing ones with a brief note on what they enable. Example:
     > "Optional: install the `context7` MCP server for faster library doc lookups. The researcher agent will fall back to web search without it."
   - Do NOT block initialization. These are recommendations, not requirements.

5. Tell the user the initialization is complete and show the created structure.
