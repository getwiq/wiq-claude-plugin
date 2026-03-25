---
name: automation
description: Automate a WIQ process end-to-end by executing each step in its blueprint using available MCP tools
---

You will receive:
- A **blueprint** containing the full automation flow, escalation paths, and tools for a WIQ process
- A **ticket/input** with its data and ID to process
- A **blurprintId** and
- A **processId** which can be used to update the execution logs
- A **tool hint** for fetching additional ticket details if needed during execution
- Additional context

## 1. Understand the blueprint before starting

Read the entire blueprint first. Identify which steps have mapped tools, which involve irreversible actions, and what the expected inputs/outputs are at each step. Understand all escalation paths — these are **global** conditions that apply throughout the entire execution, not tied to any single step (see step 2d).

## 2. Execute each node in strict sequential order

Each step is a **blocking dependency** for the next. Never skip, reorder, or parallelize steps.

**a. Identify the required tool.** Use mapped tools from `toolActions`. If `toolActions` is empty, the step is NOT optional — find another available tool (browser automation, search, API calls) or STOP and ask the user. If the required tool/connector is not available, STOP immediately and ask the user. Do NOT skip the step, do NOT move to subsequent steps, and do NOT attempt workarounds like partial outputs for later steps. A missing tool means the entire execution is blocked at this step.

**b. Execute the action.** Apply it to the ticket/input with correct parameters from ticket data and prior step outputs. On failure, retry once — if it fails again, STOP and escalate.

**c. Verify the output.** Confirm the response reflects the expected change. For irreversible actions (delete, cancel, refund, send email, status change), double-check all prior steps are verified and the data matches expectations. If anything looks off, STOP and ask the user.

**d. Verify escalation paths.** After each step, evaluate **all** escalation paths defined in the blueprint. Escalation paths are **global** — they apply throughout the entire execution, not just to a single step. Each escalation path has:
- A **trigger**: an unexpected condition or check that must be evaluated after every step (e.g. "Customer requests to speak with a manager", "SLA breach detected")
- An **action** (optional): what to do when the trigger fires. If no action is specified, the default behavior is to **stop execution and let the human supervisor intervene**.

Use the following inputs for evaluation:
- All escalation paths from the blueprint (every one, every time — they are not step-specific)
- Input and output of the steps executed so far including the current step (in blueprint order)
- The current step being executed

For every escalation path:
1. **Check the trigger against actual data.** Use concrete values from the step outputs — never assume or infer. If the data needed to evaluate a trigger is missing from the outputs, flag it as unverifiable.
2. **Classify the result:**
   - **Pass** — The trigger condition is not met. State the evidence.
   - **Escalation needed** — The trigger condition is met. State the evidence and the severity.
   - **Unverifiable** — Insufficient data to evaluate the trigger. State what is missing.
3. **Make a decision:**
   - **Approve** — All escalation paths pass. No triggers are fired and nothing is unverifiable.
   - **Reject** — One or more triggers are fired or unverifiable.

Rules for verification:
- Never approve if any escalation trigger is fired
- Never approve if any escalation path is unverifiable — lack of data is not a pass
- Base every decision on concrete evidence from the step outputs, not assumptions
- Be concise and specific in your evidence — quote actual values, IDs, and statuses

If verification rejects:
1. If a trigger fires and the escalation path has a defined **action**, execute that action (e.g. notify a specific channel, reassign the ticket) and then STOP and inform the user what happened.
2. If a trigger fires and **no action** is defined, STOP execution and let the human supervisor intervene. Report the trigger, the evidence, and ask the user how to proceed.
3. If due to lack of data (unverifiable), try to obtain the missing data. If that fails, STOP and ask the user.

**e. Capture the output.** Record relevant IDs, statuses, and values to pass to subsequent steps.

## 3. When stuck

1. Use `get_sample_instances` to see how similar tickets were handled
2. Re-read the blueprint step description and escalation paths for clues
3. If still stuck, STOP and tell the user which step, what you tried, and what's missing

## 4. Never skip and disclaim

If you cannot complete a step for any reason (missing tool, missing connector, API error, missing data), do NOT add a disclaimer and continue. Do NOT rationalize skipping it. Do NOT move to any subsequent step. Do NOT attempt partial work on later steps. STOP immediately, report what failed and why, and ask the user how to proceed. A disclaimer does not satisfy a step's requirements. Every step is a hard dependency — if step N cannot be completed, steps N+1 onward must not execute.

## 5. Summary

After completing the run, provide a summary:
- **Ticket/input processed**: ID and title/name
- **Steps completed**: List each completed step with a brief note on what was done
- **Step failed** (if any): Which step, what went wrong, what was tried
- **Escalation paths checked**: List each escalation path and the evidence produced
- **Total execution result**: Success / Partial (stopped at step N) / Failed

## 6. Update execution log
Always report the summary log using `upload_execution_log` tool with the required arguments. 