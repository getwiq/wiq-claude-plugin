# WIQ Plugin for Claude

A Claude plugin that connects [WIQ](https://www.getwiq.ai/) process intelligence with Claude's agentic capabilities, enabling end-to-end workflow automation powered by real process data.

## What is WIQ?

WIQ is an AI-powered process intelligence platform that turns invisible processes into measurable AI impact. It observes how teams actually work across tools and systems, automatically discovers hidden workflows, and generates implementation-ready AI agent specs. Learn more at [getwiq.ai](https://www.getwiq.ai/).

## What This Plugin Does

This plugin exposes WIQ's process data as MCP (Model Context Protocol) tools for Claude, allowing Claude to:

- **Discover processes** available in your WIQ workspace
- **Retrieve automation blueprints** that define step-by-step workflows with escalation paths and tool requirements
- **Execute workflows end-to-end** using available MCP tools (Linear, Slack, GitHub, etc.)
- **Validate tool readiness** before attempting automation

## Getting Started

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- Access to a WIQ workspace with configured processes

### Installation

Clone this repository and install the plugin:

```bash
claude plugin add /path/to/wiq-claude-plugin
```

The plugin connects to the WIQ API via the MCP server configuration in `.mcp.json`.

## ⚡ Commands

### 🤖 `/wiq <Describe your process>`

Orchestrates end-to-end process automation. The command will:

- Find the named process in WIQ
- Retrieve approved blueprints
- Validate that all required MCP tools are connected
- Fetch matching tickets/inputs
- Execute the blueprint steps with escalation paths verification

### 🔧 `/wiq-check-tools <Describe your process>`

Checks whether all MCP tools required by a process blueprint are available and properly configured before you attempt automation. The command:

- Identifies the process from your description
- Lists all blueprints for that process
- Lets you select one or multiple blueprints to validate
- Reports missing, unavailable, or disconnected tools
- Provides setup suggestions for publicly available tools (Linear, Slack, GitHub)

### 🧪 `/wiq-test <Describe your process>`

A trial run mode that executes a process one ticket at a time with user confirmation after every step. Use this to safely test automation before committing to full runs. The command:

- Follows the same discovery and setup as `/wiq` (process lookup, blueprint selection, tool validation)
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
