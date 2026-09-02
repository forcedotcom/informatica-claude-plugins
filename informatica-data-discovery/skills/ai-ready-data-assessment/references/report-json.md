# Report JSON payload

Copy `assets/report-template.html` verbatim. Substitute only the JSON inside `<script id="report-data" type="application/json">`. Do not alter the surrounding CSS or JS. The template is the executive scorecard (Informatica navy/orange hero, headline /100, factor heatmap, 5-level measure ladder, failure scenarios, top opportunities, projection, metadata) — the same section order as the reference HTML in `ai-readiness-report/ai-readiness-initial.html`.

The template renders:

1. **Headline number** — minimum of scored-factor maturity points (0/25/50/75/100 ladder), computed in the template. Never hand-write it into JSON.
2. **Overall status** — from the weakest measure (including 0%), shown next to maturity. This is the verdict, not the headline number. Unverified only if no catalog query ran.
3. **Factor heatmap** — Discoverability, Quality, Grounded, Accessible, Governed, Provable/Trust, Observability. Accessible is always “not in evidence” unless this skill later adds a real-time-access measure.
4. **10-measure ladder** — not 12. Do not invent Semantic grounding or Real-time access rows.
5. **Explainability table** — value, count, and what was counted. Never a formula column.
6. **Failure scenarios, top 5 opportunities, projection, follow-ons** — from the optional arrays below.

Report Observability is **catalog freshness-SLA**, not telemetry. Telemetry/tracing/monitoring stay out of the score; the method “Evidence boundary” box already says so.

## Payload shape

```json
{
  "meta": {
    "title": "AI-Ready Data Assessment",
    "heroTitle": "Support / Service Agent — Readiness Report",
    "kicker": "AI-Ready Data Assessment · Informatica CDGC",
    "useCase": "Support / Service Agent",
    "scope": "<what was in scope>",
    "mode": "Connected (CDGC)",
    "assessmentId": "<optional ARD-YYYYMM-…>",
    "runBy": "<optional>",
    "estate": "<optional short estate line if not using data.estate>",
    "generatedAt": "<ISO 8601 timestamp, now>",
    "collectedAt": "<ISO 8601 timestamp of the evidence pull>"
  },
  "overall": {
    "status": "Not Ready | Partially Ready | Ready (Illustrative) | Unverified",
    "drivingMeasure": "<measure name>",
    "note": "<one sentence: why that measure set the ceiling>"
  },
  "estate": "<530-style one-liner: sources · objects · glossary · policies from THIS run>",
  "whyItCaps": "<plain-language why the driving measure caps this use case>",
  "badges": ["Weakest-link scoring", "7 factors · 10 measures", "<scope crumb>", "Use case: Support / Service Agent"],
  "kpis": [
    { "label": "<short>", "value": "<display>", "foot": "<one line>" }
  ],
  "method": {
    "fromCdgc": "<what this session read>",
    "fromUser": "<AI use case plus A, D, U and anything else attested>",
    "scoring": "<optional override; note Support / Service Agent emphasis on Discoverability · Governed · Observability. Overall status is still the weakest measure.>",
    "boundary": "<optional override>"
  },
  "actions": [
    {
      "title": "<imperative, specific>",
      "measure": "<measure it lifts>",
      "factor": "<factor name>",
      "move": "<e.g. Move L1 (Absent) → L4 (Managed) · Current 0.0014% · Target ≥ 75>",
      "action": "<what to do>",
      "detail": "<fallback if action is omitted>",
      "why": "<why it caps this use case>",
      "capability": "<Informatica product/capability>",
      "lift": "<optional, e.g. +12 — only if you can defend it without a formula in JSON>",
      "ceilingNote": "<optional short ceiling side-effect>"
    }
  ],
  "scenarios": [
    {
      "measure": "<measure name>",
      "factor": "<factor name>",
      "valueLabel": "<Observed 0.0014% | Unverified | …>",
      "maturity": "<Absent | Ad hoc | Unverified | …>",
      "prompt": "<question an agent would be asked, grounded in THIS catalog>",
      "agent": "<confident wrong answer the gap would allow>",
      "reality": "<what the catalog evidence actually implies>",
      "why": "<why the metric matters>"
    }
  ],
  "projection": {
    "headline": "<optional facts-row, e.g. Applying the top 5 lifts the score from 0 to 50 (Defined).>",
    "note": "<optional callout under actions>",
    "rows": [
      { "metric": "<name>", "current": "<now>", "after": "<after moves>", "delta": "<+n or —>" }
    ]
  },
  "followOn": ["<plain-language follow-on after the ranked moves>"],
  "inputs": [
    { "label": "AI use case", "value": "Support / Service Agent", "note": "user-supplied (default)" },
    { "label": "Freshness SLA window", "value": "15 days", "note": "user-supplied (default 15)" }
  ],
  "measures": [
    {
      "id": "catalog-coverage",
      "name": "Catalog coverage",
      "value": 0,
      "valueLabel": "Observed|Customer-attested|Illustrative|Estimated|Post-remediation|Unverified",
      "confidence": "High|Medium|Low|Unverified",
      "numerator": 0, "denominator": 0, "unit": "catalog sources",
      "scope": "<what this counted>",
      "explain": "<one sentence for the ladder + explainability table; not a formula>",
      "evidenceSource": "CDGC Catalog Discovery — search_assets",
      "collectedAt": "<ISO timestamp>",
      "limitations": "<plain-language limitation>",
      "remediation": "<what would move this>"
    }
  ]
}
```

For a measure whose **query returned no result** (`total_matches` 0, empty results, NL 403 after retry, 502 after backoff, or denominator 0), set `"value": 0` with `"valueLabel": "Observed"` and put the empty-query reason in `limitations`. Do **not** use `"value": null` / Unverified for that case. A 0% measure is included in factor means and the headline ladder (Absent). `scenarios` are required for 0% and any measure below 40%.

## Fill rules

- Never add a `formula` field and never write a formula string anywhere in the JSON or the HTML. If asked to "show the math," point to [formulas.md](formulas.md).
- Never write a headline score, factor score, or readiness index into the JSON. The template computes those from `measures`.
- Every measure must carry a value label from `Observed`, `Customer-attested`, `Illustrative`, `Estimated`, `Post-remediation`, or `Unverified` — never a bare number with no label.
- Policy, classification, and lineage are `Observed` when this session's NL (or KEYWORD) calls completed — including when they returned 0 matches. Never carry over from a prior run. If NL returns 0 or stays 403 after the skill's retry rules, they are **0%** (`value: 0`, `Observed`), not Unverified.
- Golden-record match rate is the dedicated skill's `C / T` on `core.numValuesProfiled`. Label it `Observed` when those counts succeeded. Put in `limitations` that CDGC exposes profiling-attribute coverage, not MDM match accuracy.
- Order `measures` exactly as measures 1–10 run them. The template re-sorts the heatmap by factor.
- `explain` is a one-sentence count story (`14 of 22 in-scope sources last scanned inside the 15-day window`). Do not paste ratio algebra.
- `kpis` is four tiles from **this** run (typical: sources registered / in-scope N, sources within SLA, one governance or classification gap, one unstructured or glossary fact). Do not reuse another report’s numbers.
- `actions` is up to **5** by impact, each naming the measure (and factor) it lifts. Prefer the `action` / `why` / `capability` fields so the rec cards match the executive layout.
- `meta.useCase` and `meta.heroTitle` are **Support / Service Agent** (the only implemented use case). Do not write another use-case name into the payload. Do **not** set `meta.tenant` — the report does not show a Tenant row.
- `scenarios` are required for every measure whose value is **0%** or below 40%. Frame them as a support / service agent (customer questions, ticket triage, order + warranty + install-base) grounded in **this catalog’s** source names, scan dates, and counts. Never copy Meridian / CX-4471 / Support-desk demo copy.
- `inputs` carries one entry per user-supplied answer (`AI use case`, `A`, `D`, `U`) — quote the value with its unit and mark it `user-supplied` (or `default`) in `note`. For `D`, note if default 15 or which preset/custom was used. Do **not** ask for or store `A_class`.
- Optional `factorTargets` maps factor id → target points (defaults: 85, Observability 90, Provable/Trust 86).
- Write the finished file to `artifacts/cdgc-ai-ready-assessment-<UTC timestamp, YYYYMMDD-HHMMSS>.html`. `artifacts/` is untracked — do not commit it.
