# Changelog — wiq

All notable changes to the `wiq` plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- `/wiq-blueprint-from-skill <skill name>` — read the content of another named skill and create a WIQ blueprint from it (via the `create_blueprint` MCP tool), grounding the blueprint's steps, escalation paths, tools, and ticket-matching rules in the skill's content

## [0.1.0] - 2026-03-23

### Added

- MCP server configuration for WIQ Staging API (`.mcp.json`)
- **Skills**
  - `/wiq <process>` — orchestrate end-to-end process automation: find the process, select an approved blueprint, fetch matching tickets, and execute the workflow inline
  - `/wiq-check-tools <process>` — interactive tool readiness check: identify the process, select one or more blueprints, then validate required MCP tools inline
  - `/wiq-test <process>` — trial run mode: execute a process one ticket at a time with user confirmation after every step, for safely testing automation before full runs
- Plugin metadata via `.claude-plugin/plugin.json`
