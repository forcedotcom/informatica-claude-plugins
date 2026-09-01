# Overall status from the weakest measure

The gate grouping below is an **internal sanity check only**. It exists so a weak measure can't be averaged away — it is never rendered in the report, never named in the report, and has no field in the report JSON.

| Measure | Internal gate |
|---|---|
| Catalog coverage | Discoverability |
| Unstructured index | Discoverability |
| Metadata completeness | Discoverability |
| Glossary coverage | Groundedness |
| Classification measure | Governed |
| Policy coverage | Governed |
| Data-quality score | Quality |
| Golden-record match rate (proxy) | Quality / Provable-Trust |
| Catalog Freshness-SLA compliance | Freshness |
| Lineage coverage | Provable/Trust |

**Observability has no measure in this set** — none of the ten CDGC queries touch telemetry, tracing, or monitoring evidence. The template's standing disclaimer already says so; repeat it once in the chat reply. Never invent a score for it, and never re-add a gate section to carry it.

Overall status is set by the **weakest measure**, never an average:

- **Not Ready** when at least one measure is undefined or sits far below the rest — name that measure and its number as the reason.
- **Partially Ready** when every measure is counted and the weakest is materially short of target but not floor-level.
- **Ready (Illustrative)** only when no single measure sets a floor, and always carrying the Illustrative caveat.
- **Unverified** when the evidence needed to make the call never returned.

Say which measure drove the call, and its number, in plain language — that is the report's headline, not a footnote. Golden-record's true accuracy claim is always `Unverified` regardless of the proxy ratio.
