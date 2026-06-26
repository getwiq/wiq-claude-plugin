---
name: wiq-check-tools
description: Validate that all required MCP tools are available for a WIQ blueprint
disable-model-invocation: false
---

Help the user validate tool readiness for automating a WIQ blueprint.

## Step 1: Find the blueprint

If "$ARGUMENTS" is provided, use it as the blueprint or process name. Otherwise, ask the user which blueprint they want to validate.

Use the `list_blueprints` MCP tool to fetch available blueprints and match the user's input. If the match is ambiguous, show the list and ask the user to pick one.

## Step 2: Select blueprints

- If there is only one blueprint, use it automatically and skip to Step 3.
- If there are multiple blueprints, present the list (with their rules/descriptions) and ask the user to select one or more. Include an "All Blueprints" option. The user can select multiple blueprints.

## Step 3: Validate tools

For each selected blueprint ID, validate tool readiness directly in this session. Do not delegate to a subagent.

1. Use the `get_blueprint_details` MCP tool to fetch the full automation flow, escalation paths, and tools.
2. Extract the apps/tools required at each node in the blueprint. Include mapped tools from `toolActions` and any app/tool requirements described in the step text.
3. Compare those requirements against the MCP tools and servers available in this session.
4. Classify each required tool:
   - **Available** — the required tool is present and appears usable.
   - **Missing** — no available MCP tool can satisfy the requirement.
   - **Not connected** — the app/server appears configured, but the relevant tool is unavailable, unauthenticated, or returning connection/auth errors.
   - **Mapping mismatch** — the blueprint names a tool/app that differs from the available MCP tool that should be used.
5. Report results per blueprint:
   - Blueprint name and ID
   - Overall readiness: ready / not ready
   - Each blueprint step that requires a tool
   - Required app/tool for that step
   - Availability classification
   - Evidence from the available tools or observed error
6. Give setup suggestions for missing tools:
   - For public tools such as Linear, Slack, GitHub, or browser automation, provide concise setup guidance.
   - For internal or custom tools, tell the user they need access from their team or admin.
7. If a blueprint tool mapping does not match the available MCP tools, ask the user for confirmation before changing it. Only after confirmation, use the `update_blueprint` tool to update the blueprint with the correct mappings.

Stop after reporting validation results. Do not execute the blueprint or mutate tickets.
