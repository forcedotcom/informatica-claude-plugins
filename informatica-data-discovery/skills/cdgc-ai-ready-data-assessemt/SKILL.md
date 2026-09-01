---
name: cdgc-ai-ready-data-assessemt
description: >-
  Runs a full CDGC-connected AI-Ready Data Assessment in one pass: catalog
  coverage, catalog freshness-SLA compliance, metadata completeness, policy
  coverage, unstructured index, classification measure, data-quality score,
  golden-record match rate, glossary coverage, and lineage coverage. Asks for
  every required user input once, derives overall readiness from the weakest
  measure, and writes a self-contained HTML scorecard under artifacts/.
  TRIGGER when: user asks for an AI-ready data assessment, a CDGC readiness
  scorecard, a combined CDGC measures report, or any of the ten measures
  together. DO NOT TRIGGER when: the user wants a single unrelated catalog
  lookup, a source-system query, or an Informatica knowledge-base article.
metadata:
  version: "2.0"
  domains: ["CDGC"]
  relatedSkills:
    - "cdgc-catalog-coverage"
    - "cdgc-catalog-freshness-sla"
    - "cdgc-metadata-completeness-measure"
    - "cdgc-policy-coverage"
    - "cdgc-unstructured-index"
    - "cdgc-classification-measure"
    - "cdgc-data-quality-score"
    - "cdgc-golden-record-match-rate"
    - "cdgc-glossary-coverage"
    - "cdgc-lineage-coverage"
---

# cdgc-ai-ready-data-assessemt: AI-Ready Data Assessment

A single **connected-mode** pass over CDGC that produces every readiness measure this skill tracks, then renders one HTML scorecard. It replaces running ten separate measures one at a time.

## When This Skill Owns the Task

Use `cdgc-ai-ready-data-assessemt` when the work involves:
- a full AI-ready / CDGC readiness scorecard
- all ten measures together (coverage, freshness, completeness, policy, unstructured, classification, DQ, golden-record proxy, glossary, lineage)
- writing `artifacts/cdgc-ai-ready-assessment-*.html`

Delegate elsewhere when the user is:
- asking a one-off catalog question (churn, a table, a glossary term) → Catalog Discovery MCP directly
- querying source data (Oracle/Snowflake) → the relevant SQL connection
- looking up Informatica product docs → Informatica knowledge MCP

---

## Required Context to Gather First

Need, in this order:
1. CDGC Catalog Discovery MCP (`user-catalog-discovery`) ready.
2. A live `IDS-SESSION-ID` on that MCP. If any search fails or the session has expired, ask for a new session ID before continuing — do not guess or reuse a stale dump.
3. After query-derived numbers are known, four non-negative integers in **one** batch (do not default them):
   - `A` — additional catalog sources still needing a scan
   - `D` — freshness SLA window, in days
   - `U` — unstructured assets known but not yet in CDGC
   - `A_class` — additional classifications still to add

This is read-only CDGC evidence. Nothing here proves source-system access enforcement, complete discovery, or source-data correctness — say so in the report, don't imply it.

---

## Recommended Workflow

### 1. Discover catalog-source topology once
Read [references/topology.md](references/topology.md). Build `T` (all primary sources) and `N` (object count > 1). Reuse that origin list for coverage, freshness, completeness, golden-record, and lineage.

### 2. Compute all ten measures from live CDGC
Read [references/measures.md](references/measures.md) and run measures 1–10 against Catalog Discovery (`search_assets` / `get_asset_details`). Do not reuse a prior session's dump.

### 3. Batch the four user inputs
Stop once and ask for `A`, `D`, `U`, and `A_class` as non-negative integers. If any answer is missing, ask again for just that value — do not proceed to computation until all four are numeric.

### 4. Compute internally
Read [references/formulas.md](references/formulas.md). Never put a formula in the report JSON or the HTML.

### 5. Set overall status from the weakest measure
Read [references/overall-status.md](references/overall-status.md). Never average. Observability is not in this evidence set — say so once in chat; never invent a score for it.

### 6. Render the HTML scorecard
Copy [assets/report-template.html](assets/report-template.html) verbatim. Substitute only the JSON inside `#report-data`. Payload shape and fill rules: [references/report-json.md](references/report-json.md). Write to `artifacts/cdgc-ai-ready-assessment-<UTC YYYYMMDD-HHMMSS>.html`. Do not commit `artifacts/`.

### 7. Reply in chat
Keep it short: overall status, the driving measure and its number, that observability is not measured, and the path to the HTML file. Do not paste the ten measure tables, the JSON, or any formula. Do not quote the readiness index as the verdict.

---

## High-Signal Rules

- Never reuse a stale catalog dump; if search fails, ask for a new `IDS-SESSION-ID`.
- Ask for `A`, `D`, `U`, and `A_class` once, together; never assume defaults.
- Overall status = weakest counted measure, never an average of the ten.
- Golden-record is an Unverified profiling proxy, not match accuracy.
- Policy coverage is `Observed` only when this session's cheap `search_assets` PDP check succeeded.
- Undefined measure → `value: null`, never `0`.
- No `formula` field in JSON. No readiness-gate section in the report.
- Never write the readiness index into JSON; the template computes it.
- `get_asset_details` segment enum is US `neighborhood`; retry if UK spelling is rejected.

---

## Output Format

When finishing, report in this order:
1. **Overall status** (Not Ready / Partially Ready / Ready (Illustrative) / Unverified)
2. **Driving measure** and its number
3. **Observability** — not measured by this evidence set
4. **Path** to the generated HTML file
5. One line each for any measure that came back Unverified because a query returned nothing

Suggested shape:

```text
Status: <status>
Driven by: <measure> at <value>
Observability: not in this evidence set
Report: artifacts/cdgc-ai-ready-assessment-<timestamp>.html
```

---

## Reference File Index

| File | When to read |
|------|-------------|
| `references/topology.md` | Building the shared primary-origin list (`T`, `N`) |
| `references/measures.md` | Query logic for measures 1–10 |
| `references/formulas.md` | Internal ratios and rounding — never render |
| `references/overall-status.md` | Weakest-measure status and internal gates |
| `references/report-json.md` | JSON payload shape and fill rules |
| `assets/report-template.html` | Verbatim HTML shell; substitute `#report-data` only |
