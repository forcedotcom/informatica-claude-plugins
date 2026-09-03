# Internal formulas

Never render formulas in the HTML report or in the report JSON. If asked to "show the math," point here instead of putting it in the report. Each ratio matches the dedicated measure skill.

```
catalog_coverage          = N / (N + A) * 100
freshness_sla             = S / N * 100
metadata_completeness     = E / T_meta * 100
policy_coverage           = R_pol / T_pol * 100
unstructured_index        = C_unstr / (C_unstr + U) * 100
classification_measure    = C_cls / T_cls * 100
data_quality_score        = Q / D * 100
golden_record_match_rate  = C_profiled / T_col * 100
glossary_coverage         = G / T_gloss * 100
lineage_coverage          = L / T_lineage * 100
```

Guard every division: if a denominator is 0 **because a query returned no result**, that measure is **0%** (`value: 0`), not undefined. Same when a numerator query returns 0 matches or NL does not return a result (403 after retry, 502 after backoff).

Dedicated skills that say “cannot be counted / undefined” for empty NL are **overridden** in this assessment — report **0%**:

- Policy: NL `all objects related to policy` returns 0, or NL is disabled (403 after retry) → `R_pol = 0`, coverage **0%** (still report `T_pol` if it counted).
- Classification: NL `data sets and data elements related to classification` returns 0 **with the concrete 12-class filter AND unfiltered** → `C_cls = 0`, measure **0%** (still report `T_cls`). A 0 from the filtered call alone is not enough: if the unfiltered NL total is higher, the class list is incomplete — fix the list and recompute rather than reporting 0%.
- Lineage: NL lineage returns 0 after retries, or NL is disabled after lineage retry rules → `L = 0`, coverage **0%** (still report `T_lineage`). Do not treat a 502 as 0 until backoff retries are exhausted.
- Metadata: batched NL returns 0 for **both** business name and classifications (or NL 403) → `E = 0`, completeness **0%** (still report `T_meta`).
- Data quality: zero RuleInstances → `D = 0`, `Q = 0`, score **0%**.
- Unstructured: `C_unstr + U = 0` → **0%**. `C_unstr = 0` with `U > 0` is also **0%**.
- Catalog coverage: `N + A = 0` → **0%**.
- Freshness: `N = 0` → **0%**.
- Glossary: `T_gloss = 0` → **0%**.
- Golden-record: `T_col = 0` (or profiling aggregation returned no result) → **0%**.

Do not use KEYWORD `*` as a substitute for an NL relationship query. Count the empty NL as 0.

Round to 2 decimal places unless the value is a whole number. If the result is below 0.10%, report 4 decimal places.
