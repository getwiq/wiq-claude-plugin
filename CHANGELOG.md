# Changelog — wiq

All notable changes to the `wiq` plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.1.0] - 2026-03-23

### Added

- MCP server configuration for WIQ Staging API (`.mcp.json`)
- **Agents**
  - `automation` — execute blueprint steps for a single ticket/input with guardrail checks, tool validation, retry logic, and structured summary reporting
  - `tool-validator` — validate required MCP tools against blueprint steps, reporting available/missing/not-connected tools per blueprint with setup suggestions
- **Skills**
  - `/automate <process>` — orchestrate end-to-end process automation: find the process, select a blueprint, fetch matching tickets, and launch automation agents (sequential or parallel)
  - `/tool-validation <process>` — interactive tool readiness check: identify the process, select one or more blueprints (multi-select with "All Blueprints" option), then delegate validation to the tool-validator agent
- Plugin metadata via `.claude-plugin/plugin.json`