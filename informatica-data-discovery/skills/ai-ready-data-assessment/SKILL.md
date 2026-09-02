---
name: ai-ready-data-assessment
description: >-
  Runs a full CDGC-connected AI-Ready Data Assessment in one pass: catalog
  coverage, catalog freshness-SLA compliance, metadata completeness, policy
  coverage, unstructured index, classification measure, data-quality score,
  golden-record match rate, glossary coverage, and lineage coverage. Asks first
  which AI use case to score against (only Support / Service Agent is
  implemented; default that if omitted). Then asks for A, D, and U once,
  derives overall readiness from the weakest measure, and writes a
  self-contained executive HTML scorecard under artifacts/. TRIGGER when: user
  names ai-ready-data-assessment, cdgc-ai-ready-assessment, or the former
  catalog-ai-ready-data-assessment / cdgc-ai-ready-data-assessemt,
  asks for an AI-ready data assessment, a CDGC readiness scorecard, a combined
  CDGC measures report, or any of the ten measures together. DO NOT TRIGGER
  when: the user wants a single unrelated catalog lookup, a source-system
  query, or an Informatica knowledge-base article.
metadata:
  version: "4.4"
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

# ai-ready-data-assessment: AI-Ready Data Assessment

A single **connected-mode** pass over CDGC that produces every readiness measure this skill tracks, then renders one executive HTML scorecard (headline /100, factor heatmap, 5-level measure ladder, failure scenarios, top opportunities). Query grain and formulas come from the ten dedicated measure skills — do not use older blended or PDP-path definitions. **This assessment** reports **0%** (not undefined) when a query returns no result. Do not copy another tenant’s demo narrative into the report.

## When This Skill Owns the Task

Use `ai-ready-data-assessment` when the work involves:
- a full AI-ready / CDGC readiness scorecard
- all ten measures together
- writing `artifacts/cdgc-ai-ready-assessment-*.html`

Delegate elsewhere when the user is:
- asking a one-off catalog question → Catalog Discovery MCP directly
- querying source data → the relevant SQL connection
- looking up Informatica product docs → Informatica knowledge MCP
- asking for **one** of the ten measures alone → that dedicated skill

---

## Required Context to Gather First

Need, in this order:
1. **AI use case** — ask first, using [references/use-case.md](references/use-case.md). Default is **Support / Service Agent**. Other options are not supported yet: say so and ask the user to choose Support / Service Agent. Do not query CDGC until option 1 is accepted.
2. CDGC Catalog Discovery MCP (`user-catalog-discovery`) ready.
3. A live `IDS-SESSION-ID` on that MCP. If any search fails or the session has expired, ask for a new session ID before continuing — do not guess or reuse a stale dump.
4. After query-derived numbers are known, **three** inputs in **one** batch:
   - `A` — additional catalog sources still needing a scan (non-negative integer; **no default**)
   - `D` — freshness SLA window in days — **default 15**; offer **7**, **15**, **30**, or **custom**
   - `U` — unstructured assets known but not yet in CDGC (non-negative integer; **no default**)

Do **not** ask for `A_class`. Classification measure no longer uses additional-classifications-to-add.

This is read-only CDGC evidence. Nothing here proves source-system access enforcement, complete discovery, or source-data correctness — say so in the report, don't imply it.

---

## Recommended Workflow

### 0. Select the AI use case (stop here until accepted)
Read [references/use-case.md](references/use-case.md). Ask the picker **before** topology or any catalog search. Default: **Support / Service Agent**. If the user chooses RAG / Knowledge Assistant, Analytics / BI Copilot, Autonomous / Action Agent, or types something else, reply that it is not supported yet and ask them to choose Support / Service Agent. Do not proceed until that is selected.

### 1. Discover catalog-source topology once
Read [references/topology.md](references/topology.md). Build `T_src` and `N` with **batched** Published origin counts (≤10 origins per aggregation). Reuse that origin list for coverage, freshness, completeness, and golden-record. Lineage and classification are class-type grain, not that origin list — and both must use **concrete** class types, never abstract `core.DataSet` / `core.DataElement` as a `classType` filter (see the class-type trap in [references/measures.md](references/measures.md)).

### 2. Compute all ten measures from live CDGC
Read [references/measures.md](references/measures.md) and run measures 1–10. Do not reuse a prior session's dump. Match each dedicated skill's call budget (do not fan out per origin or per RuleInstance when the skill forbids it).

### 3. Batch the three user inputs
Stop once and ask for `A`, `D`, and `U`. For `D`, present 7 / 15 (default) / 30 / custom. If `D` is omitted or “default”, use **15**. If `A` or `U` is missing, ask again for just that value — do not proceed until `A` and `U` are numeric.

### 4. Compute internally
Read [references/formulas.md](references/formulas.md). Never put a formula in the report JSON or the HTML.

### 5. Set overall status from the weakest measure
Read [references/overall-status.md](references/overall-status.md). Never average. Telemetry observability (tracing / monitoring) is not in this evidence set — say so once in chat; never invent a score for it. Catalog freshness-SLA **is** the Observability factor in the HTML.

### 6. Render the executive HTML scorecard
Copy [assets/report-template.html](assets/report-template.html) verbatim. Substitute only the JSON inside `#report-data`. Payload shape and fill rules: [references/report-json.md](references/report-json.md). The template must stay visually aligned with the executive report (hero, headline /100, KPI strip, method boxes, factor heatmap, measure ladder, explainability without formulas, failure scenarios, top 5 opportunities, projection, metadata). Write to `artifacts/cdgc-ai-ready-assessment-<UTC YYYYMMDD-HHMMSS>.html`. Do not commit `artifacts/`.

### 7. Reply in chat
Keep it short: overall status, the driving measure and its number, that telemetry observability is not measured, and the path to the HTML file. Do not paste the ten measure tables, the JSON, formulas, or run-metrics tables. Do not quote the headline number as the verdict — status is the weakest measure.

---

## High-Signal Rules

- Never reuse a stale catalog dump; if search fails, ask for a new `IDS-SESSION-ID`.
- Ask the AI use case **first**. Only Support / Service Agent is implemented; default to it if omitted. Other choices → not supported yet; re-ask for option 1. Do not query CDGC until accepted.
- Ask for `A`, `D`, and `U` once together. `D` may default to 15. Never ask for `A_class`.
- Overall status = weakest counted measure (including **0%**), never an average of the ten.
- Policy coverage is `R_pol / T_pol` (objects related to policy over Published Data Element + Data Set + Classification) — **not** a 100% PDP live-path check.
- Classification is `C_cls / T_cls` over the **concrete 12-class** data-set + data-element list (Column, ViewColumn, Calculation, ResultSetColumn, FlatField, `core.DataElement`, Table, View, ResultSet, ExternalTable, FlatFile, `core.DataSet`) — **not** a blended column/inventory average, and **never** abstract `core.DataSet` / `core.DataElement` as a `classType` filter, which forces `C_cls = 0`. Same list on both calls. Before reporting `C_cls = 0`, re-run the NL query unfiltered; a higher total means the class list is incomplete, not that the tenant has no classifications.
- Unstructured `C_unstr` is Published `UnstructuredFile` only — **not** all objects on object-store `resourceType`s.
- Freshness uses `lastScannedOn` only — **not** `lastFullScanOn`.
- Glossary `T` / `G` are **Published** BusinessTerm + Metric only — **not** Draft.
- Lineage is Published `core.DataSet` + `core.DataElement`; skip neighbourhood; do not send `aggregationSpec` on NL.
- If **any query does not return a result** (`total_matches` 0, empty `results`, or NL 403 after retry / 502 after backoff), that count is **0** and the measure is **0%**, not undefined. Do not use `value: null` / Unverified for an empty or failed query. Dedicated skills that say “cannot be counted / undefined” are overridden here.
- `value: 0` with `valueLabel: "Observed"` for a 0% measure. Put in `limitations` that the query returned no matches (or NL was disabled). Still report the other side of the ratio when it exists.
- No `formula` field in JSON. No formula column in the HTML. No readiness-gate section.
- Never write the headline score, factor scores, or a readiness index into JSON; the template computes them.
- Do not add Semantic grounding or Real-time access measures. Accessible stays “not in evidence.”
- Do not set `meta.tenant` or render a Tenant row (no “Tenant · Connected-mode CDGC”).
- Failure scenarios must use this catalog’s sources and counts — never copy a demo tenant’s story.
- `actions` is up to 5. `scenarios` are required for any measure at **0%** or below 40%.
- `get_asset_details` segment enum is US `neighborhood`; retry if UK spelling is rejected.
- Do not dump dedicated-skill **Run metrics** tables into the assessment chat or HTML.

---

## Output Format

When finishing, report in this order:
1. **Overall status** (Not Ready / Partially Ready / Ready (Illustrative) / Unverified)
2. **Driving measure** and its number
3. **Telemetry observability** — tracing and monitoring are not measured by this evidence set (catalog freshness-SLA is scored)
4. **Path** to the generated HTML file
5. One line each for any measure that is **0%** because a query returned no matches or NL did not return a result

Suggested shape:

```text
Status: <status>
Driven by: <measure> at <value>
Telemetry observability: not in this evidence set
Report: artifacts/cdgc-ai-ready-assessment-<timestamp>.html
```

---

## Reference File Index

| File | When to read |
|------|-------------|
| `references/use-case.md` | First question: AI use case picker (only Support / Service Agent proceeds) |
| `references/topology.md` | Building the shared primary-origin list (`T_src`, `N`) |
| `references/measures.md` | Query logic for measures 1–10 (aligned to dedicated skills) |
| `references/formulas.md` | Internal ratios and rounding — never render |
| `references/overall-status.md` | Weakest-measure status and factor map |
| `references/report-json.md` | JSON payload shape and fill rules |
| `assets/report-template.html` | Verbatim HTML shell; substitute `#report-data` only |
