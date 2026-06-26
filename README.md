# WIQ Plugin for Claude and Codex

A Claude and Codex plugin that connects [WIQ](https://www.getwiq.ai/) process intelligence with agentic coding assistants, enabling end-to-end workflow automation powered by real process data.

## What is WIQ?

WIQ is an AI-powered process intelligence platform that turns invisible processes into measurable AI impact. It observes how teams actually work across tools and systems, automatically discovers hidden workflows, and generates implementation-ready AI agent specs. Learn more at [getwiq.ai](https://www.getwiq.ai/).

## What This Plugin Does

This plugin exposes WIQ's data as MCP (Model Context Protocol) tools for Claude and Codex, allowing an agent to:

- **Retrieve automation blueprints** that define step-by-step workflows with escalation paths and tool requirements
- **Search past work** to see how users handled similar entities and tasks across applications
- **Learn from prior executions** to see how other agents ran the same blueprint
- **Execute workflows end-to-end** using available MCP tools (Linear, Slack, GitHub, etc.)
- **Validate tool readiness** before attempting automation

## Getting Started

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or [Codex](https://developers.openai.com/codex) installed
- Access to a WIQ workspace with configured blueprints

### Installation

Clone this repository and install the plugin for the host you use.

For Claude:

```bash
claude plugin add /path/to/wiq-claude-plugin
```

For Codex, add this repository as a marketplace source, then install the `wiq` plugin from that marketplace:

```bash
codex plugin marketplace add /path/to/wiq-claude-plugin
codex plugin add wiq@wiq
```

The plugin connects to the WIQ API via the shared MCP server configuration in `wiq/.mcp.json`. The WIQ MCP server handles OAuth discovery and registration, so the plugin does not commit a fixed OAuth client ID.

### Plugin Layout

- `wiq/.mcp.json` — shared MCP server config for Claude and Codex
- `wiq/.claude-plugin/plugin.json` — Claude plugin manifest
- `wiq/.codex-plugin/plugin.json` — Codex plugin manifest
- `.claude-plugin/marketplace.json` — Claude marketplace metadata
- `.agents/plugins/marketplace.json` — Codex marketplace metadata

## Commands

Claude users can invoke these as slash commands. Codex users can invoke the WIQ plugin or the corresponding bundled skill by name.

### `/wiq <Describe your task>`

Runs end-to-end automation inline in your session — one ticket at a time, sequentially. Step execution stays in the main conversation (no subagent handoff) so you can observe progress and interject between steps. The command will:

- Find matching approved blueprints in WIQ
- Validate that all required MCP tools are connected
- Fetch matching tickets/inputs and let you choose which to process
- Execute the blueprint steps for each ticket with escalation path verification

For parallel processing, run `/wiq` in a separate session — this skill is sequential by design.

### `/wiq-check-tools <Describe your task>`

Checks whether all MCP tools required by a blueprint are available and properly configured before you attempt automation. The command:

- Lists available blueprints matching your description
- Lets you select one or multiple blueprints to validate
- Reports missing, unavailable, or disconnected tools
- Provides setup suggestions for publicly available tools (Linear, Slack, GitHub)

### `/wiq-try <Describe your task>`

A trial run mode that executes a blueprint one ticket at a time with user confirmation after every step. Use this to safely test automation before committing to full runs. The command:

- Follows the same discovery and setup as `/wiq` (blueprint selection, tool validation)
- Processes tickets sequentially — never in parallel
- Pauses after every step for user confirmation before proceeding
- Verifies escalation paths with evidence from actual step outputs
- Stops immediately on failures or missing tools — no skipping or disclaimers
- Provides a detailed summary of each ticket's execution result

## Key Design Principles

- **Sequential execution** — Blueprint steps run in strict order with no skipping
- **Evidence-based Escalation Paths** — Conditions are verified against real data, not assumptions
- **Fail-safe behavior** — Automation stops and escalates on tool unavailability or escalation path violations
- **Structured reporting** — Clear summaries of actions taken and outcomes

## License

© Copyright 2026 WIQ by Bardeen Inc. All rights reserved.
