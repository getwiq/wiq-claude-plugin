---
description: Trial run of a WIQ process — executes one ticket at a time with user confirmation after every step
disable-model-invocation: false
---

**This is a trial run.** You must ask the user for confirmation after every step before proceeding to the next. No parallel execution is allowed — tickets are always processed one at a time, sequentially.

Automate the WIQ process "$ARGUMENTS" as a trial run.

## Phase 1: Discovery and setup

1. Use the `list_processes` MCP tool to find the process matching "$ARGUMENTS" and get its ID
2. Use the `list_blueprints` tool with the process ID to get available blueprints and their ticket matching rules
3. From the blueprints, identify:
   - What the input/ticket type is (e.g. Linear ticket, GitHub issue, support request, etc.)
   - Which tool is needed to fetch those inputs (e.g. `list_issues`, `search_issues`, etc.)
4. Select the correct blueprint based on what the user wants to automate, ask the user if ambiguous then use `get_blueprint_details` to fetch the full automation flow, escalation paths, and tools
5. Quick tool validation: check the blueprint's required tools/apps against your available MCP tools. If any are missing, tell the user which tools they need to connect before proceeding. Do not continue until all required tools are available.
6. Use the appropriate MCP tool to find the relevant tickets/inputs that match the blueprint's criteria
7. Ask the user which tickets/inputs they want to process (all, a subset, or a specific one)

## Phase 2: Execute each ticket sequentially

For each selected ticket/input, execute the blueprint steps directly (do NOT delegate to an agent). Process one ticket at a time, never in parallel.

Read the entire blueprint first. Identify which steps have mapped tools, which involve irreversible actions, and what the expected inputs/outputs are at each step. Each step needs to be verified for escalation paths (see step 2d).

### Execute each node in strict sequential order

Each step is a **blocking dependency** for the next. Never skip, reorder, or parallelize steps.

**a. Identify the required tool.** Use mapped tools from `toolActions`. If `toolActions` is empty, the step is NOT optional — find another available tool (browser automation, search, API calls) or STOP and ask the user. If the required tool/connector is not available, STOP immediately and ask the user. Do NOT skip the step, do NOT move to subsequent steps, and do NOT attempt workarounds like partial outputs for later steps. A missing tool means the entire execution is blocked at this step.

**b. Execute the action.** Before executing, present the user with the action you are about to take, the tool you will use, and the parameters you will pass. Wait for the user to confirm before proceeding. Once confirmed, apply it to the ticket/input with correct parameters from ticket data and prior step outputs. On failure, retry once — if it fails again, STOP and escalate.

**c. Verify the output.** Confirm the response reflects the expected change. For irreversible actions (delete, cancel, refund, send email, status change), double-check all prior steps are verified and the data matches expectations. If anything looks off, STOP and ask the user.

**d. Verify escalation paths.** After each step, evaluate all escalation paths defined for that step directly. Use the following inputs:
- The blueprint's escalation paths for the current step
- Input and output of the steps executed so far including the current step (in blueprint order)
- The current step being executed

For every escalation path on the current step:
1. **Check the condition against actual data.** Use concrete values from the step outputs — never assume or infer. If the data needed to evaluate a condition is missing from the outputs, flag it as unverifiable.
2. **Classify the result:**
   - **Pass** — The condition does not trigger escalation. State the evidence.
   - **Escalation needed** — The condition is met. State the evidence and the severity.
   - **Unverifiable** — Insufficient data to evaluate. State what is missing.
3. **Make a decision:**
   - **Approve** — All escalation paths pass. No conditions are triggered and nothing is unverifiable.
   - **Reject** — One or more escalation paths are triggered or unverifiable.

Rules for verification:
- Never approve if any escalation is triggered
- Never approve if any escalation path is unverifiable — lack of data is not a pass
- Base every decision on concrete evidence from the step outputs, not assumptions
- Be concise and specific in your evidence — quote actual values, IDs, and statuses

If verification rejects:
1. If due to lack of data, try to obtain the missing data. If that fails, STOP and ask the user.
2. If the escalation is valid with evidence, STOP and ask the user how to proceed based on the evidence.

**e. Capture the output.** Record relevant IDs, statuses, and values to pass to subsequent steps.

**f. Ask for user confirmation.** Present the user with:
- What was done in this step
- The output/result
- The escalation path verification decision
- What the next step will be

Wait for the user to confirm before proceeding to the next step.

### When stuck

1. Use `get_sample_instances` to see how similar tickets were handled
2. Re-read the blueprint step description and escalation paths for clues
3. If still stuck, STOP and tell the user which step, what you tried, and what's missing

### Never skip and disclaim

If you cannot complete a step for any reason (missing tool, missing connector, API error, missing data), do NOT add a disclaimer and continue. Do NOT rationalize skipping it. Do NOT move to any subsequent step. Do NOT attempt partial work on later steps. STOP immediately, report what failed and why, and ask the user how to proceed. A disclaimer does not satisfy a step's requirements. Every step is a hard dependency — if step N cannot be completed, steps N+1 onward must not execute.

## Phase 3: Summary

After completing the run for each ticket, provide a summary:
- **Ticket/input processed**: ID and title/name
- **Steps completed**: List each completed step with a brief note on what was done
- **Step failed** (if any): Which step, what went wrong, what was tried
- **Escalation paths checked**: List each escalation path and the evidence produced
- **Total execution result**: Success / Partial (stopped at step N) / Failed

### Update execution log
Always report the summary log using `upload_execution_log` tool with the required arguments. 