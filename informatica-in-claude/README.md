# informatica-in-claude

Trusted, catalog-grounded data discovery for Informatica IDMC in Claude Code.

This plugin targets two Informatica IDMC capability areas — **CDGC** (Cloud Data Governance and Catalog) and **Claire** (Informatica's GenAI/AI engine). Today, CDGC is covered by one real, substantive skill (catalog discovery).

Install via the `claude-plugins-official` marketplace.

## Quick Start

1. **Add the marketplace.**
   ```
   /plugin marketplace add claude-plugins-official
   ```
2. **Install the plugin.**
   ```
   /plugin install informatica-in-claude@claude-plugins-official
   ```
3. **Restart Claude Code / reload plugins** to pick up the manifest, MCP server, and skills.

## Sample Prompts

The `catalog-and-data-discovery` skill activates automatically on data questions like:

- Where's the customer data with the highest quality?
- Can I rely on the RATING field in FCT_ORDERS?
- Is this data safe to use for a marketing campaign?
- Who owns the SALES_FACT table, and is it certified?
- What policy applies to this PII column?
- Is there a right-to-be-forgotten flag on this dataset before I run outreach?

## Verify, Update, and Uninstall the Plugin

- **Verify:** Confirm the plugin is loaded by checking that `catalog-and-data-discovery` appears in Claude Code's Skills list and that `informatica-catalog-discovery` and `informatica-data-exploration` appear in the active MCP servers.
- **Update:** `/plugin marketplace update claude-plugins-official`, reinstall/upgrade the plugin, and then restart Claude Code / reload plugins.
- **Uninstall:** `/plugin uninstall informatica-in-claude@claude-plugins-official` and then restart.

## What's Included

### 1 Skill (CDGC)

| Area | Skill | Description |
|------|-------|--------------|
| Discovery & trust | `catalog-and-data-discovery` | Intent-driven catalog discovery. Discovers assets, assembles tenant context (glossary, ownership, certification, ratings), assesses trust/reliability per field, checks sensitivity/PII, looks up applicable policy, gives safe-usage guidance, and checks compliance flags (RTBF/consent) with structured, severity-ranked gap reporting. Grounded exclusively in catalog metadata; zero-hallucination rules and a verdict-first response format. |

### What Else is in the Box

- **MCP servers:** Two entries in `.mcp.json` (`type: "http"`): `informatica-catalog-discovery` (CDGC) and `informatica-data-exploration` (Claire).
- **Plugin manifest:** `.claude-plugin/plugin.json` declares `name`, `version` (`1.0.0`), `description`, and `author` only. There's no `userConfig` field, so there's no install-time prompt for a tenant/environment host.
