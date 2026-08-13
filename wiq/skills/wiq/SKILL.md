---
name: wiq
description: Automate a WIQ process end-to-end
disable-model-invocation: false
---

Your goal: figure out what the user wants to automate, find the right APPROVED blueprint for it, identify the tickets/inputs to process, and execute the blueprint **inline** for each ticket — one ticket at a time, sequentially. Do not delegate execution to a subagent; staying in the main session keeps the user in the loop and able to interject between steps.

Automate the WIQ process "$ARGUMENTS" end-to-end.

## Phase 1: Discovery and setup

1. Use the `list_blueprints` MCP tool to find approved blueprints matching "$ARGUMENTS". Each row carries its Starting Point `triggers[]` (`title`, `source`, `onlyUse`, `exclude`, `runOnceForEach`) — that's what tells two similar-sounding blueprints apart, so use it to choose. Rows have no `toolActions` or `content`; those are platform-specific and come from `get_blueprint_details` in the next step.
2. Select the correct blueprint based on what the user wants to automate, ask the user if ambiguous, then use `get_blueprint_details` once to fetch the full automation flow, escalation paths, and tools. If the response includes a `customization` field, it contains YOUR user's personal instructions for this blueprint (e.g. which segment or customers they handle) — apply them to every run. Reference this fetched detail throughout — do not re-fetch per ticket.
3. **Read `flow.start` — the Starting Point.** This is the blueprint's authoritative definition of what starts a run and how to find pending work. It is NOT a step, never appears in `flow.steps`, and is excluded from the `1..N` step numbering. Each entry in `flow.start.triggers[]` gives you:
   - `title` — the trigger's name; use it to label this channel's candidates
   - `source` — the inbound channel and object in human terms (always present)
   - `runOnceForEach` — the unit of ONE execution, i.e. what you iterate over (always present)
   - `onlyUse` — the qualifying condition: an item must match this to be in scope
   - `exclude` — the disqualifying condition: refuse an item matching this, even if it also matches `onlyUse` (exclude wins)
   - `toolActions` — the `Tool.method` entries that list this trigger's pending work, resolving against the blueprint's `tools[]`. Usually one, but a channel can need several calls (e.g. one to page a list, another to read each record) — run all of them for that trigger. May be empty, in which case fall back to the tool paths in step 4.
   - `content` — optional prose blocks recording how this trigger behaves on THIS platform: polling cadence, a filter the connector can't express, a field the API omits. Read it before calling `toolActions`; it's usually where the reason your call returns something unexpected is written down.

   `flow.start.toolActions` lists the trigger-source apps across the whole Starting Point, and `flow.start.isAutomatable` says whether finding the work can be done unattended.

   Do not infer the input type from the description or step 1 when `flow.start` is present — it is more specific and authoritative. `flow.start` is absent on a blueprint that has no Starting Point yet; only then fall back to inferring from the description and step 1.
4. **Establish the execution path up front.** Check the blueprint's required tools/apps against the tools available in this session *now* (don't assume from past sessions). For each missing tool, walk this list **once, top to bottom, and stop at the first path that's available** — never re-offer a path you've already ruled out:
   1. **MCP connector** — *if* the app appears in the connector registry, offer to connect it (most reliable). Not in the registry, or the user declines → go to 2.
   2. **Browser extension** (Claude in Chrome) — if usable, use it. Otherwise → go to 3.
   3. **Computer use / screen control** — last resort.
   If none of the three applies, STOP and ask the user. Settle every required tool's path before executing any step, and never silently probe browser tabs or screen access.
5. **Run EVERY trigger and pool the candidates.** `flow.start.triggers[]` may hold several triggers; each is a separate inbound channel, and work can arrive on any of them. Do not pick one, and do not stop after the first that returns results:

   a. For each trigger, call every entry in its `toolActions` to list pending work, honoring anything its `content` says about how that channel behaves.
   b. Filter each trigger's results by its OWN `onlyUse` / `exclude` — the conditions are per-trigger, not shared.
   c. **Apply `customization` as a further narrowing.** When `customization` is present, apply it on top of each trigger's `onlyUse` / `exclude` — never as a replacement. The blueprint's conditions define what the process handles; the user's instructions narrow that to what THEY handle (e.g. "only EMEA dairy customers"). Keep both counts per trigger — the candidates that survived `onlyUse` / `exclude`, and the subset that also survived `customization` — so you can report the difference in the listing below.
   d. Present the surviving candidates to the user **grouped by trigger**, labelled with each trigger's `title`, so they can see where each item came from:

      ```
      Support Inbox Emails (3 candidates)
        1. "Refund request — order #8821"     — 2h ago
        2. ...
      Zendesk Tickets (1 candidate, 4 hidden by your customization)
        4. "Cancel my subscription"           — 20m ago
      ```

      Number the items continuously across groups so the user can select by number. Where `customization` excluded items, say so next to that trigger's count, so the user understands why a channel shows fewer candidates than expected.
   e. If a trigger's `toolActions` fail or its tools are unavailable, say so explicitly next to that trigger's group and continue with the others — a partial pool is usable, but the user must know a channel is missing rather than silently seeing fewer options.
   f. If a trigger returns nothing, show it with "(no pending work)". An empty channel is information, not something to hide.
   g. If the same underlying item appears under two triggers, show it once and note both trigger names.

6. Ask the user which candidates to process (all, a subset, or one). Each selected item is one run, iterating per the trigger's `runOnceForEach`. If the user wants parallel processing, tell them to run a separate session — this skill is sequential.

## Phase 2: Execute each ticket inline

For each selected ticket/input, execute the blueprint **directly in this session**. Process one ticket fully before starting the next. Never spawn a subagent for execution. Do not pause for user confirmation between steps unless you need some input/clarification or something fails or an escalation fires — this is full automation.

### 1. Understand the blueprint before starting

Read the entire blueprint first. Identify which steps have mapped tools, which involve irreversible actions, and what the expected inputs/outputs are at each step. Understand all escalation paths — these are **global** conditions that apply throughout the entire execution, not tied to any single step (see step 2d).

**Two different things are called "triggers".** `flow.start.triggers[]` are *entry* conditions — evaluated ONCE, before step 1, to find work. `escalationPaths[].trigger` are *exception* conditions — evaluated after EVERY step. Never re-evaluate a Starting Point trigger mid-run, and never treat an escalation trigger as a source of work.

### 2. Execute each node in strict sequential order

Each step is a **blocking dependency** for the next. Never skip, reorder, or parallelize steps.

**a. Identify the required tool.** Use mapped tools from `toolActions`. If `toolActions` is empty or the mapped tool is unavailable, walk these paths **once, top to bottom, and stop at the first that's available** — never re-offer a path you've ruled out: (1) **MCP connector**, only if the app is in the connector registry; (2) **browser extension**; (3) **computer use**. The moment a path is unavailable or declined, move to the next. If none applies, STOP and ask the user. Do NOT skip the step, do NOT move to subsequent steps, and do NOT attempt workarounds like partial outputs for later steps. A missing tool means the entire execution is blocked at this step.

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

### 3. When stuck

1. Use `search_past_work` with `detail: "high_level"` to see how users handled similar work — this shows task-level patterns and decision context
2. Use `search_past_work` with `detail: "browser_steps"` to see detailed step-by-step browser actions users took for similar entities
3. Use `search_past_executions` to see how other agents handled similar tickets with this blueprint — learn from prior automated runs
4. Re-read the blueprint step description and escalation paths for clues
5. If still stuck, STOP and tell the user which step, what you tried, and what's missing

For research-heavy lookups whose raw output would bloat this session, summarize the relevant findings before continuing. Step execution and research both stay inline.

### 4. Never skip and disclaim

If you cannot complete a step for any reason (missing tool, missing connector, API error, missing data), do NOT add a disclaimer and continue. Do NOT rationalize skipping it. Do NOT move to any subsequent step. Do NOT attempt partial work on later steps. STOP immediately, report what failed and why, and ask the user how to proceed. A disclaimer does not satisfy a step's requirements. Every step is a hard dependency — if step N cannot be completed, steps N+1 onward must not execute.

### 5. Summary

After completing the run for a ticket, provide a summary:
- **Ticket/input processed**: ID and title/name
- **Steps completed**: List each completed step with a brief note on what was done
- **Step failed** (if any): Which step, what went wrong, what was tried
- **Escalation paths checked**: List each escalation path and the evidence produced
- **Total execution result**: Success / Partial (stopped at step N) / Failed

### 6. Update execution log

Always report the summary log using `upload_execution_log` tool with:
- `blueprintId` — the blueprint used
- `ticketId` — the ticket/input ID
- `content` — the summary text
- `outcome` — one of: `"success"` (all steps completed), `"escalated"` (hit an escalation path and handed off), `"failed"` (encountered error or blocker), or `"unknown"` (outcome unclear)

Then mark the ticket as **closed** in the running narrative and ask user if they want to process the next ticket.
