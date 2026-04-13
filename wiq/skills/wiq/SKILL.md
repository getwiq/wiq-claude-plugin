---
description: Automate a WIQ process end-to-end
disable-model-invocation: false
---

Your goal: figure out what the user wants to automate, find the right APPROVED blueprint for it, identify the tickets/inputs to process, and hand each one off to an automation agent for execution. You are the orchestrator — you do not execute blueprint steps yourself.

Automate the WIQ process "$ARGUMENTS" end-to-end.

1. Use the `list_blueprints` MCP tool to find approved blueprints matching "$ARGUMENTS" and their ticket matching rules
2. Select the correct blueprint based on what the user wants to automate, ask the user if ambiguous then use `get_blueprint_details` to fetch the full automation flow, escalation paths, and tools
3. From the blueprint details, identify:
   - What the input/ticket type is (e.g. Linear ticket, GitHub issue, support request, etc.)
   - Which tool is needed to fetch those inputs (e.g. `list_issues`, `search_issues`, etc.)
4. Quick tool validation: check the blueprint's required tools/apps against your available MCP tools. If any are missing, tell the user which tools they need to connect before proceeding. Do not continue until all required tools are available.
5. Use the appropriate MCP tool to find the relevant tickets/inputs that match the blueprint's criteria
6. Ask the user which tickets/inputs they want to process (all, a subset, or a specific one)
7. If more than one ticket/input needs to be executed, ask the user whether to execute in parallel or sequential and follow that to launch the agents
8. For each selected ticket/input, launch a separate `wiq:automation` agent with:
   - The full blueprint details (always use only approved)
   - The ticket/input data and its ID
   - The `blueprintId` for this ticket/input that will be used by the agent
   - Which tool to use for fetching additional ticket details if needed during execution
   - Additional context needed for the agent
