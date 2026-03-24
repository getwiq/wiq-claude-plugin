---
name: automation
description: Automate a WIQ process end-to-end by executing each step in its blueprint using available MCP tools
---

You will receive:
- A **blueprint** containing the full automation flow, guardrails, and tools for a WIQ process
- A **ticket/input** with its data and ID to process
- A **tool hint** for fetching additional ticket details if needed during execution
- Additional context

## 1. Understand the blueprint before starting

Read the entire blueprint first. Identify which steps have mapped tools, which have guardrails, which involve irreversible actions, and what the expected inputs/outputs are at each step.

## 2. Execute each node in strict sequential order

Each step is a **blocking dependency** for the next. Never skip, reorder, or parallelize steps.

For each node:

**a. Check guardrails first.** If the step has an associated guardrail, produce concrete evidence the condition is satisfied — actual data retrieved via tools, not assumptions or ticket descriptions. By severity:
- **Critical**: Hard blocker. No evidence = STOP and escalate.
- **Warning**: Verify and produce evidence. If ambiguous, ask the user before continuing.
- **Info**: Check if possible, proceed with caution.

Good evidence: "Retrieved order #12345 — status 'delivered', no refunds on record"
Bad evidence: "The ticket says the order was delivered" or "Unable to check, proceeding anyway"

**b. Identify the required tool.** Use mapped tools from `toolActions`. If `toolActions` is empty, the step is NOT optional — find another available tool (browser automation, search, API calls) or STOP and ask the user.

**c. Execute the action.** Apply it to the ticket/input with correct parameters from ticket data and prior step outputs. On failure, retry once — if it fails again, STOP and escalate.

**d. Verify the output.** Confirm the response reflects the expected change. For irreversible actions (delete, cancel, refund, send email, status change), double-check all prior steps are verified and the data matches expectations. If anything looks off, STOP and ask.

**e. Capture the output.** Record relevant IDs, statuses, and values to pass to subsequent steps.

## 3. When stuck

1. Use `get_sample_instances` to see how similar tickets were handled
2. Re-read the blueprint step description and guardrails for clues
3. If still stuck, STOP and tell the user which step, what you tried, and what's missing

## 4. Never skip and disclaim

If you cannot complete a step, do NOT add a disclaimer and continue. Do NOT rationalize skipping it. STOP immediately, report what failed and why, and ask the user how to proceed. A disclaimer does not satisfy a step's requirements.

## 5. Summary

After completing the run, provide a summary:
- **Ticket/input processed**: ID and title/name
- **Steps completed**: List each completed step with a brief note on what was done
- **Step failed** (if any): Which step, what went wrong, what was tried
- **Guardrails checked**: List each guardrail and the evidence produced
- **Total execution result**: Success / Partial (stopped at step N) / Failed

Report this summary log using `upload_execution_log` tool with the required arguments. 