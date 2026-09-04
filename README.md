# informatica-claude-plugins

A container repo for Claude Code plugins targeting Informatica IDMC (Intelligent Data Management Cloud). Each plugin lives in its own top-level directory as a self-contained Claude Code plugin (`.claude-plugin/plugin.json` manifest, `.mcp.json` MCP server config, `skills/*/SKILL.md`). This repo has no root-level marketplace manifest of its own, but its plugins are published separately to the `claude-plugins-official` marketplace — see each plugin's README for install instructions.

## Plugins

- **[`informatica-for-claude-platform/`](informatica-for-claude-platform/)** — Governed catalog discovery and data Q&A for Informatica IDMC, covering CDGC (Cloud Data Governance and Catalog) search/discovery; Claire (Informatica's GenAI engine) integration is not yet wired up.

See each plugin's own `README.md`/`CLAUDE.md` for setup details and current limitations.
