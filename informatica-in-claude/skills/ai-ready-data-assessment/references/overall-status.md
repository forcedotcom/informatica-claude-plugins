# Overall status from the weakest measure

Overall **status** is set by the **weakest measure**, never an average:

- **Not Ready** when at least one measure is **0%** or sits far below the rest — name that measure and its number as the reason.
- **Partially Ready** when every measure is counted and the weakest is materially short of target but not floor-level.
- **Ready (Illustrative)** only when no single measure sets a floor, and always carrying the Illustrative caveat.
- **Unverified** only when no catalog query ran at all (session missing, MCP never reached). An empty query result is **0%**, not Unverified.

A **0%** measure (`value: 0`) is counted and can set status to **Not Ready**.

Say which measure drove the call, and its number, in plain language.

The HTML **headline number** is different: the template maps each counted measure onto the 5-level ladder (Absent 0 / Ad hoc 25 / Defined 50 / Managed 75 / Optimized 100), averages those points inside a factor, then takes the **minimum scored factor**. Do not put that number in JSON, and do not let a comfortable-looking headline soften a **Not Ready** status.

## Factor map (rendered in the report)

| Measure | Factor in the report |
|---|---|
| Catalog coverage | Discoverability |
| Unstructured index | Discoverability |
| Metadata completeness | Discoverability |
| Golden-record match rate | Quality |
| Data-quality score | Quality |
| Glossary coverage | Grounded |
| Classification measure | Governed |
| Policy coverage | Governed |
| Lineage coverage | Provable/Trust |
| Catalog Freshness-SLA compliance | Observability |

**Accessible** (real-time / API / batch) has no connected-mode measure — the template shows it as not in evidence and excludes it from the headline minimum. Do not invent a score.

**Observability in the report** means catalog freshness-SLA (`lastScannedOn` vs window `D`). It is not telemetry, tracing, or monitoring. Those remain outside this evidence set: say so once in chat; never invent a telemetry score.

Do not add Semantic grounding or Real-time access as extra measures. The ten dedicated skills are the full set.
