---
name: tool-validator
description: Validate that Claude has the required MCP tools to automate a process from its blueprint
---

Given a **process ID** and one or more **blueprint IDs**:

1. For each blueprint ID, use the `get_blueprint_details` MCP tool to fetch the full automation flow, guardrails, and tools
2. Extract the apps/tools required at each node in each blueprint
3. Check which MCP tools/servers are currently configured in this session
4. Report (per blueprint):
   - Be specific about which blueprint step requires which tool.
   - Which required tools are available
   - Which required tools are missing
   - Which required tools are not connected
   - Overall readiness (ready / not ready)
5. Suggestions for missing tools (if applicable)
   - Give instructions on how to set up these tools. If they are readily available (e.g., public MCP servers like Linear, Slack, GitHub), provide setup steps. For internal or custom tools, note that the user needs to contact their team or admin to get access.
6. Optionally, allow users to modify the tools in blueprint using `update_blueprint` if the tools needed seem to be wrong