# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This repo is a **placeholder scaffold** for a Claude Code plugin (`informatica-data-discovery`, see `.claude-plugin/plugin.json`) targeting Informatica IDMC's two capability areas:

- **CDGC** (Cloud Data Governance and Catalog)
- **Claire** (Informatica's GenAI/AI engine)

There is no build/test/lint tooling in this repo (no `package.json`, `Makefile`, or `scripts/` directory) — it's a plugin manifest, an MCP server config, and Markdown skills. There is nothing to build; validate changes by reading the JSON/Markdown directly.

## Current state of the plugin manifest

`.claude-plugin/plugin.json` declares only `name`, `version`, `description` (currently an empty string), and `author`. It has **no `userConfig` field** — there is currently no mechanism for Claude Code to prompt an installer for a tenant/environment host at install or enable time.

`.mcp.json` declares a single server, `cdgc-catalog-discovery` (`type: "http"`), with `url: ""` — a literal empty string, not a `${user_config.*}` substitution and not a hardcoded host. As written, this entry is non-functional until a real URL is supplied, either as a literal value in `.mcp.json` or by adding a `userConfig` field to `plugin.json` and referencing it here (e.g. `${user_config.cdgc_mcp_host}`).

There is **no `claire-data-management-orchestrator` entry, and no Claire-related server at all**, in `.mcp.json`. The Claire capability area currently has no MCP wiring in this plugin — do not assume a Claire server exists until one is added to `.mcp.json`.

There is no `.claude-plugin/marketplace.json` in this repo, so it cannot be installed via `/plugin marketplace add` against itself; it can only be added as a local plugin path today.

Do not implement business logic against real CDGC/Claire APIs without explicit direction — this repo intentionally has none yet.

## Skills

- `skills/catalog-discovery/SKILL.md` — real, substantive guidance for CDGC catalog discovery (intent-driven workflow covering discovery, trust/reliability, sensitivity, policy, safe-usage, and compliance flags). Not a placeholder.
- `skills/cdgc-ai-ready-data-assessemt/` — real skill that runs a full CDGC-connected AI-Ready Data Assessment across ten measures (coverage, freshness SLA, metadata completeness, policy coverage, unstructured index, classification, data quality, golden-record match rate, glossary coverage, lineage coverage) in one pass, then renders a self-contained HTML scorecard to `artifacts/` (not committed). Backed by `references/*.md` (query logic, formulas, status rules, JSON payload shape) and `assets/report-template.html`. Overall status is always the weakest measure, never an average.
