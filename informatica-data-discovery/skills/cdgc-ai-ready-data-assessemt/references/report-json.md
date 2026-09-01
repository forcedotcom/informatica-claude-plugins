# Report JSON payload

Copy `assets/report-template.html` verbatim. Substitute only the JSON inside `<script id="report-data" type="application/json">`. Do not alter the surrounding CSS or JS.

Overall readiness is the headline. The template renders:

1. The **status** from the weakest measure, as the large text.
2. The **readiness index** in the gauge — the unweighted mean of the counted measure values, computed by the template itself. Never hand-write it, never treat it as the verdict, and never let a comfortable-looking index soften the status: a 47% index alongside a `Not Ready` status is the correct, intended output.
3. The **strength distribution** — how many measures landed Strong (≥80%), Moderate (40–79%), Weak (<40%), or Unverified (uncounted).

## Payload shape

```json
{
  "meta": {
    "title": "AI-Ready Data Assessment",
    "useCase": "<use case being assessed>",
    "scope": "<what was in scope, e.g. connected-mode CDGC tenant>",
    "mode": "Connected (CDGC)",
    "generatedAt": "<ISO 8601 timestamp, now>",
    "collectedAt": "<ISO 8601 timestamp of the evidence pull>"
  },
  "overall": {
    "status": "Not Ready | Partially Ready | Ready (Illustrative) | Unverified",
    "drivingMeasure": "<measure name>",
    "note": "<one sentence: why that measure set the ceiling>"
  },
  "actions": [
    { "title": "<imperative, specific>", "detail": "<what it moves and roughly how far>", "measure": "<measure it lifts>" }
  ],
  "inputs": [
    { "label": "Freshness SLA window", "value": "15 days", "note": "user-supplied" }
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
      "evidenceSource": "CDGC Catalog Discovery — search_assets",
      "collectedAt": "<ISO timestamp>",
      "limitations": "<plain-language limitation>",
      "remediation": "<what would move this>"
    }
  ]
}
```

For an **undefined** measure (denominator 0, or the query returned nothing), set `"value": null` with `"valueLabel": "Unverified"` and put the reason in `limitations`. The template then shows a dash instead of a number, leaves the bar empty, and excludes it from the readiness index — which is why you must never substitute `0` for undefined. Omit `actions` entirely rather than padding with filler.

## Fill rules

- Never add a `formula` field and never write a formula string anywhere in the JSON. If asked to "show the math," point to [formulas.md](formulas.md).
- Every measure must carry a value label from `Observed`, `Customer-attested`, `Illustrative`, `Estimated`, `Post-remediation`, or `Unverified` — never a bare number with no label.
- Golden-record match rate's `valueLabel` is always `Unverified` (or `Estimated` at best) with the proxy caveat in `limitations`, regardless of how clean the underlying ratio looks.
- Policy coverage's `valueLabel` is `Observed` only when this session's live PDP check succeeded — never carried over from a prior run.
- Never write a readiness index into the JSON. The template computes it from the measures it can count.
- `actions` is the top 3 by impact, each naming the measure it lifts.
- `inputs` carries one entry per user-supplied answer — quote the value with its unit and mark it `user-supplied` in `note`.
- Order `measures` exactly as measures 1–10 run them. The cards render in array order.
- Write the finished file to `artifacts/cdgc-ai-ready-assessment-<UTC timestamp, YYYYMMDD-HHMMSS>.html`. `artifacts/` is untracked — do not commit it.
