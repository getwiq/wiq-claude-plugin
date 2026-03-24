---
description: Validate that all required MCP tools are available for a WIQ process
disable-model-invocation: false
---

Help the user validate tool readiness for automating a WIQ process.

## Step 1: Identify the process

If "$ARGUMENTS" is provided, use it as the process name. Otherwise, ask the user which process they want to validate.

Use the `list_processes` MCP tool to fetch available processes and match the user's input. If the match is ambiguous, show the list and ask the user to pick one.

## Step 2: Select blueprints

Use the `list_blueprints` MCP tool with the selected process ID to fetch all available blueprints.

- If there is only one blueprint, use it automatically and skip to Step 3.
- If there are multiple blueprints, present the list (with their rules/descriptions) and ask the user to select one or more. Include an "All Blueprints" option. The user can select multiple blueprints.

## Step 3: Validate tools

Once you have the process ID and the selected blueprint ID(s), use the `tool-validator` agent to run the validation. Pass it the **process ID** and the **blueprint ID(s)** the user selected (or all blueprint IDs if they chose "All Blueprints").

Report the agent's findings back to the user.
