# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`informatica-claude-plugins` is a container repo for Claude Code plugins targeting Informatica IDMC. It currently holds a single plugin, `informatica-for-claude-platform/`, structured as a standard Claude Code plugin directory (`.claude-plugin/plugin.json` manifest, `.mcp.json` MCP server config, `skills/*/SKILL.md`).

There is **no root-level `.claude-plugin/marketplace.json`** — this repo is not itself a plugin marketplace (`/plugin marketplace add` won't work against the repo root). Its plugin(s) are, however, published separately to the external `claude-plugins-official` marketplace, which is the recommended install path (see `informatica-for-claude-platform/README.md`'s Quick Start).

There is **no build/test/lint tooling anywhere** in this repo (no `package.json`, `Makefile`, `scripts/`, or CI workflows) — plugins here are pure manifest/config/Markdown. Validate changes by reading the JSON/Markdown directly; there is nothing to run.

## informatica-for-claude-platform plugin

Targets two Informatica IDMC capability areas:
- **CDGC** (Cloud Data Governance and Catalog)
- **Claire** (Informatica's GenAI/AI engine)

Current state (see `informatica-for-claude-platform/CLAUDE.md` for full detail):
- `.mcp.json` declares two servers, `informatica-catalog-discovery` (CDGC) and `informatica-data-exploration` (Claire), both `type: "http"` with a literal empty `url` — non-functional until real endpoints are supplied. There is no `userConfig` field in `plugin.json` to prompt an installer for a tenant host, and no `${user_config.*}` substitution wired in.
- `skills/catalog-and-data-discovery/SKILL.md` is a real, substantive skill (not a placeholder): an intent-driven catalog discovery workflow covering discovery, trust/reliability assessment, sensitivity/PII classification, policy lookup, safe-usage guidance, and compliance-flag (RTBF/consent) checks with structured gap detection, plus a hand-off to the `data-exploration-agent` MCP for reading actual data.

Do not implement business logic against real CDGC/Claire APIs without explicit direction — this scaffold intentionally has none yet.

## Adding another plugin

Follow the same shape as `informatica-for-claude-platform/`: a top-level directory with its own `.claude-plugin/plugin.json`, `.mcp.json` (if it needs MCP servers), `skills/*/SKILL.md`, and its own `CLAUDE.md`/`README.md`. If this repo is later turned into a marketplace source itself, a root `.claude-plugin/marketplace.json` listing each plugin directory will be needed; today, new plugins would instead be published to the external `claude-plugins-official` marketplace like `informatica-for-claude-platform/` is.
