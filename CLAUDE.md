# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`informatica-claude-plugins` is a container repo for Claude Code plugins targeting Informatica IDMC. It currently holds a single plugin, `informatica-data-discovery/`, structured as a standard Claude Code plugin directory (`.claude-plugin/plugin.json` manifest, `.mcp.json` MCP server config, `skills/*/SKILL.md`).

There is **no root-level `.claude-plugin/marketplace.json`** — this repo is not yet set up as an installable plugin marketplace (`/plugin marketplace add` won't work against the repo root). Each plugin subdirectory is self-contained and would need to be added individually as a local plugin path.

There is **no build/test/lint tooling anywhere** in this repo (no `package.json`, `Makefile`, `scripts/`, or CI workflows) — plugins here are pure manifest/config/Markdown. Validate changes by reading the JSON/Markdown directly; there is nothing to run.

## informatica-data-discovery plugin

Targets two Informatica IDMC capability areas:
- **CDGC** (Cloud Data Governance and Catalog)
- **Claire** (Informatica's GenAI/AI engine)

Current state (see `informatica-data-discovery/CLAUDE.md` for full detail):
- `.mcp.json` declares one server, `cdgc-catalog-discovery` (`type: "http"`), with a literal empty `url` — non-functional until a real endpoint is supplied. There is no `userConfig` field in `plugin.json` to prompt an installer for a tenant host, and no `${user_config.*}` substitution wired in.
- There is **no Claire MCP server entry at all** — do not assume Claire tooling exists until one is added to `.mcp.json`.
- `skills/catalog-discovery/SKILL.md` and `skills/cdgc-ai-ready-data-assessemt/SKILL.md` are real, substantive skills (not placeholders): an intent-driven catalog discovery workflow covering discovery, trust/reliability assessment, sensitivity/PII classification, policy lookup, safe-usage guidance, and compliance-flag (RTBF/consent) checks with structured gap detection, and a ten-measure CDGC AI-Ready Data Assessment that renders a self-contained HTML scorecard to `artifacts/` (not committed) using `references/*.md` for query/formula/status logic.

Do not implement business logic against real CDGC/Claire APIs without explicit direction — this scaffold intentionally has none yet.

## Adding another plugin

Follow the same shape as `informatica-data-discovery/`: a top-level directory with its own `.claude-plugin/plugin.json`, `.mcp.json` (if it needs MCP servers), `skills/*/SKILL.md`, and its own `CLAUDE.md`/`README.md`. If the repo is later turned into a marketplace, a root `.claude-plugin/marketplace.json` listing each plugin directory will be needed.
