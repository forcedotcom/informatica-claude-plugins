# informatica-claude-plugins

A container repo for Claude Code plugins targeting Informatica IDMC (Intelligent Data Management Cloud). Each plugin lives in its own top-level directory as a self-contained Claude Code plugin (`.claude-plugin/plugin.json` manifest, `.mcp.json` MCP server config, `skills/*/SKILL.md`). There is no root-level marketplace manifest yet, so plugins here are not installable via `/plugin marketplace add` against this repo — each one would need to be added individually as a local plugin path.

## Plugins

- **[`informatica-data-discovery/`](informatica-data-discovery/)** — Data discovery for Informatica IDMC, covering CDGC (Cloud Data Governance and Catalog) search/discovery and a ten-measure CDGC AI-Ready Data Assessment; Claire (Informatica's GenAI engine) integration is not yet wired up.

See each plugin's own `README.md`/`CLAUDE.md` for setup details and current limitations.
