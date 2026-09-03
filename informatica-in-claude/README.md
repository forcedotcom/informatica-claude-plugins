# informatica-in-claude

Trusted, catalog-grounded data discovery for Informatica IDMC in Claude Code.

This plugin targets two Informatica IDMC capability areas — **CDGC** (Cloud Data Governance and Catalog) and **Claire** (Informatica's GenAI/AI engine). Today, CDGC is covered by two real, substantive skills (catalog discovery and an AI-Ready Data Assessment scorecard).

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

The `catalog-discovery` skill activates automatically on data questions like:

- Where's the customer data with the highest quality?
- Can I rely on the RATING field in FCT_ORDERS?
- Is this data safe to use for a marketing campaign?
- Who owns the SALES_FACT table, and is it certified?
- What policy applies to this PII column?
- Is there a right-to-be-forgotten flag on this dataset before I run outreach?

For a full readiness scorecard, the `ai-ready-data-assessment` skill activates on prompts like:

- Run a CDGC AI-ready data assessment for this catalog.
- Give me a readiness scorecard across coverage, freshness, policy, and lineage.

## Verify, Update, and Uninstall the Plugin

- **Verify:** Confirm the plugin is loaded by checking that `catalog-discovery` and `ai-ready-data-assessment` appear in Claude Code's Skills list and that `cdgc-catalog-discovery` appears in the active MCP servers.
- **Update:** `/plugin marketplace update claude-plugins-official`, reinstall/upgrade the plugin, and then restart Claude Code / reload plugins.
- **Uninstall:** `/plugin uninstall informatica-in-claude@claude-plugins-official` and then restart.

## What's Included

### 2 Skills (CDGC)

| Area | Skill | Description |
|------|-------|--------------|
| Discovery & trust | `catalog-discovery` (v2.4.2) | Intent-driven catalog discovery. Discovers assets, assembles tenant context (glossary, ownership, certification, ratings), assesses trust/reliability per field, checks sensitivity/PII, looks up applicable policy, gives safe-usage guidance, and checks compliance flags (RTBF/consent) with structured, severity-ranked gap reporting. Grounded exclusively in catalog metadata; zero-hallucination rules and a verdict-first response format. |
| Readiness scorecard | `ai-ready-data-assessment` (v4.4) | Runs all ten CDGC AI-readiness measures in one connected pass — coverage, freshness SLA, metadata completeness, policy coverage, unstructured index, classification, data quality, golden-record match-rate proxy, glossary coverage, and lineage coverage — then renders a self-contained HTML scorecard to `artifacts/` (not committed). Overall status is always the weakest measure, never an average. Formulas and query logic live in `references/*.md`. The report is built from `assets/report-template.html`. |

### What Else is in the Box

- **MCP servers:** One entry, `cdgc-catalog-discovery` (`.mcp.json`, `type: "http"`).
- **Plugin manifest:** `.claude-plugin/plugin.json` declares `name`, `version` (`1.0.0`), `description`, and `author` only. There's no `userConfig` field, so there's no install-time prompt for a tenant/environment host.

## More Information

- The exact query logic, scoring formulas, and status thresholds behind the `ai-ready-data-assessment` scorecard are documented in `skills/ai-ready-data-assessment/references/*.md` (`measures.md`, `formulas.md`, `overall-status.md`, `report-json.md`, `topology.md`). Read these if you need to audit or extend how a measure's status is derived.
