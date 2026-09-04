# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This repo is a **placeholder scaffold** for a Claude Code plugin (`informatica-for-claude-platform`, see `.claude-plugin/plugin.json`) targeting Informatica IDMC's two capability areas:

- **CDGC** (Cloud Data Governance and Catalog)
- **Claire** (Informatica's GenAI/AI engine)

There is no build/test/lint tooling in this repo (no `package.json`, `Makefile`, or `scripts/` directory) — it's a plugin manifest, an MCP server config, and Markdown skills. There is nothing to build; validate changes by reading the JSON/Markdown directly.

## Current state of the plugin manifest

`.claude-plugin/plugin.json` declares only `name`, `version`, `description` (currently an empty string), and `author`. It has **no `userConfig` field** — there is currently no mechanism for Claude Code to prompt an installer for a tenant/environment host at install or enable time.

`.mcp.json` declares two servers — `informatica-catalog-discovery` (CDGC) and `informatica-data-exploration` (Claire) — both `type: "http"` with `url: ""`, a literal empty string, not a `${user_config.*}` substitution and not a hardcoded host. As written, neither entry is functional until a real URL is supplied, either as a literal value in `.mcp.json` or by adding a `userConfig` field to `plugin.json` and referencing it here (e.g. `${user_config.cdgc_mcp_host}`).

There is no `.claude-plugin/marketplace.json` in this repo, so it cannot be installed via `/plugin marketplace add` against itself; it can only be added as a local plugin path today.

Do not implement business logic against real CDGC/Claire APIs without explicit direction — this repo intentionally has none yet.

## Skills

- `skills/catalog-and-data-discovery/SKILL.md` — real, substantive guidance for CDGC catalog discovery (intent-driven workflow covering discovery, trust/reliability, sensitivity, policy, safe-usage, and compliance flags), plus a hand-off to the `data-exploration-agent` MCP for reading actual data. Not a placeholder.
