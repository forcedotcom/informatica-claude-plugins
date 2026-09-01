# informatica-data-discovery

A Claude Code plugin scaffold for Informatica IDMC, targeting two capability areas (with real Skills for CDGC):

- **CDGC** (Cloud Data Governance and Catalog) — data catalog search, governance, asset lineage.
- **Claire** — Informatica's GenAI/AI engine for AI-assisted recommendations, metadata enrichment, and natural-language data discovery.

## Status

This is a **scaffold / placeholder** repo.

- `.mcp.json` declares one MCP server, `cdgc-catalog-discovery` (`type: "http"`), with an empty `url` — it needs a real endpoint before it will work.
- There is no MCP server entry for Claire yet.
- `.claude-plugin/plugin.json` has no `userConfig` field, so there is currently no install-time prompt for a tenant/environment host. A real URL must either be hardcoded into `.mcp.json`, or a `userConfig` field must be added to `plugin.json` with a matching `${user_config.*}` substitution.

The `skills/catalog-discovery/SKILL.md` and `skills/cdgc-ai-ready-data-assessemt/SKILL.md` skills, however, are real, substantive guidance for CDGC — not placeholders. See `CLAUDE.md` for more on filling in the remaining gaps.

## Directory Structure

```
.claude-plugin/plugin.json            Plugin manifest (name, version, description, author)
.mcp.json                             MCP server config — cdgc-catalog-discovery (url not yet set)
skills/catalog-discovery/SKILL.md     CDGC catalog-discovery skill (real, not a placeholder)
skills/cdgc-ai-ready-data-assessemt/  CDGC AI-Ready Data Assessment skill (real, not a placeholder)
CLAUDE.md                             Guidance for future Claude Code sessions working in this repo
```

## Installing Locally

There is no marketplace manifest and no install/uninstall scripts in this repo. To use this plugin locally:

1. Clone this repo.
2. In Claude Code, add it as a local plugin — point your plugin config at this directory, or place/symlink it under your Claude Code plugins directory.
3. Fill in a real `url` for `cdgc-catalog-discovery` in `.mcp.json` (there is no install-time host prompt yet).
4. Restart Claude Code / reload plugins to pick up the manifest and skills.
