# Changelog — wiq

All notable changes to the `wiq` plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.1.0] - 2026-03-23

### Added

- MCP server configuration for WIQ Staging API (`.mcp.json`)
- **Agents**
  - `automation` — execute blueprint steps for a single ticket/input with escalation path verification, tool validation, retry logic, and structured summary reporting
  - `tool-validator` — validate required MCP tools against blueprint steps, reporting available/missing/not-connected tools per blueprint with setup suggestions
- **Skills**
  - `/wiq <process>` — orchestrate end-to-end process automation: find the process, select an approved blueprint, fetch matching tickets, and launch automation agents
  - `/wiq-check-tools <process>` — interactive tool readiness check: identify the process, select one or more blueprints, then delegate validation to the tool-validator agent
  - `/wiq-test <process>` — trial run mode: execute a process one ticket at a time with user confirmation after every step, for safely testing automation before full runs
- Plugin metadata via `.claude-plugin/plugin.json`