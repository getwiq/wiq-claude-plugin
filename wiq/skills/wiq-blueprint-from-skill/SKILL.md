---
name: wiq-blueprint-from-skill
description: Read the content of another named skill and create a WIQ blueprint from it. Use when the user wants to turn a Claude/Codex skill into a WIQ blueprint, "import a skill into WIQ", "make a blueprint from the <name> skill", or "create a blueprint from a skill".
disable-model-invocation: false
---

Turn the named skill "$ARGUMENTS" into a WIQ blueprint by reading that skill's content and calling the `create_blueprint` MCP tool. The skill is the source of truth — every blueprint step must come from something the skill actually says. Do not invent steps, tools, or rules the skill doesn't describe.

## Phase 1: Read the source skill

Read the content of the skill named in "$ARGUMENTS" — its `SKILL.md` and any files it bundles (reference docs, templates, examples), since those often hold the real procedure detail. You're reading it to describe a workflow, so don't run it or any script it bundles.

- If "$ARGUMENTS" is empty, ask the user which skill to convert.
- If no such skill exists or the name is ambiguous, say so and ask the user to pick from the skills you can see — never guess.

## Phase 2: Map the skill to a blueprint

Read the whole skill first, then translate it into the `create_blueprint` argument shape. Ground each field in the skill text.

- **name** — a short, human-readable blueprint name derived from the skill's purpose (e.g. the skill's `name`/title in title case, or a clearer phrasing of what it automates).
- **description** — one or two sentences describing what the workflow accomplishes, from the skill's description and opening intent.
- **flow.steps** — the ordered procedure. Walk the skill's instructions top to bottom and emit one step per meaningful action, in execution order:
  - `number` — 1-based, sequential.
  - `title` — a short imperative phrase for the action ("Fetch the ticket", "Post the summary to Slack").
  - `content` — an array of content blocks capturing the step's detail. Use blocks:
    - `{ "type": "paragraph", "text": "..." }` for prose.
    - `{ "type": "numbered-list", "items": ["...", "..."] }` for ordered sub-steps.
    - `{ "type": "bullet-list", "items": ["...", "..."] }` for unordered detail.
    - `{ "type": "sub-path", "label": "If X", "content": [ ...blocks ] }` for conditional branches.
    Preserve the decision rules, expected inputs/outputs, and verification checks the skill describes.
  - `isAutomatable` — `true` if a tool or agent can perform the step unattended; `false` if it inherently needs a person. When `false` (or when the skill says to confirm/review with a human), set `hitlCheckpoint` to a short description of what the person must decide.
  - `toolActions` — for each app/tool the step uses, a `"toolName.methodName"` string that matches an entry in the `tools` array (below). Empty array if the step uses no external tool.
- **flow.escalationPaths** — the skill's stop/fail/hand-off conditions become escalation paths (they are global, not tied to one step). Any "STOP and ask the user", "escalate", "if X fails", "if you can't do Y" instruction maps to:
  - `trigger` — the condition (e.g. "Required tool is unavailable", "Refund amount exceeds $500").
  - `action` — what to do when it fires, if the skill specifies one (e.g. "Notify #ops and reassign"). Omit `action` when the skill's intent is simply to stop and hand off to a human.
  - `toolActions` — any `"toolName.methodName"` the action needs.
- **tools** — the apps/tools the flow references. For each: `name`, `description`, and `methods[]` where each method is `{ name, description, inputs: [{name,type,required?}], outputs: [{name,type}], status }`. `toolActions` strings in steps/escalations must be `"tool.name.method.name"` combinations that exist here.
  - **Cross-check each method against the tools actually available in this session** — the connected MCP servers/tools and available shell commands — the same way `/wiq-check-tools` validates a blueprint. Set `status`:
    - `"available"` — a real MCP tool or shell command in this session can satisfy the method. Prefer that real tool's actual name for the method `name` (and the matching `toolActions` string) when it differs from what the skill calls it, so the blueprint maps onto tools that exist.
    - `"unavailable"` — the skill needs this capability but no connected tool or command provides it (missing, not connected, or unauthenticated).
    - `"unknown"` — only when you genuinely can't tell whether it's satisfiable.
  - Do not invent tools the skill doesn't reference, and do not drop a capability the skill needs just because it's `"unavailable"` — record it as `"unavailable"` so the gap is visible.
- **ticketMatching** — if the skill implies what it runs on (a Linear ticket, a support email, an invoice, etc.), set `categories` (short labels) and `primaryEntityType` (the canonical entity, e.g. `"ticket"`, `"invoice"`), and a `condition` string if the skill only applies under specific circumstances. If the skill doesn't imply an input type, pass `{ "categories": [] }`.
- **optimizedFor** — set it to the harness you are running in: `"claude"` if you are Claude (Claude Code / Claude), or `"openai-codex"` if you are Codex. The skill is authored and tool-checked against your session, so this records the agent platform the blueprint was built for.

Guidelines:
- Keep steps faithful and self-contained: someone reading only the blueprint should be able to follow the skill's procedure.
- Don't collapse distinct actions into one step, and don't split a single action into many. One meaningful action per step.
- If the skill is thin or ambiguous on a point, say so rather than inventing detail — ask the user or leave the step description honest about the gap.

## Phase 3: Confirm, then create

1. Present a concise preview of the mapped blueprint: name, description, the numbered step titles (with which are non-automatable / have HITL checkpoints), the tools with their availability (`available` / `unavailable` / `unknown` from the cross-check above), and the ticketMatching. Call out any `unavailable` tools as gaps the user will need to connect before the blueprint can run, and note anything you couldn't determine from the skill.
2. Ask the user to confirm or adjust. Apply any requested changes.
3. Call the `create_blueprint` MCP tool with the mapped arguments (`name`, `description`, `flow`, and `tools` / `ticketMatching` when non-empty).
4. Report the result:
   - The returned `blueprintId`.
   - That it was created as a **private draft** owned by the caller.
   - Next steps: refine it with the `update_blueprint` tool if needed, and submit it for approval in the WIQ app.
5. If `create_blueprint` fails, report the error and what to fix (e.g. a malformed `flow`); do not retry blindly.
