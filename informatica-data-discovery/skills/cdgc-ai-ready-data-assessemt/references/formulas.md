# Internal formulas

Never render formulas in the HTML report or in the report JSON. If asked to "show the math," point here instead of putting it in the report.

```
catalog_coverage          = N / (N + A) * 100
freshness_sla             = S / N * 100
metadata_completeness     = E / T_meta * 100
policy_coverage           = 100   (only after a live PDP confirmation)
unstructured_index        = C_unstr / (C_unstr + U) * 100
classification_measure    = ((Cc / Tc) * 100 + (R / (K + A_class)) * 100) / 2
data_quality_score        = Q / D * 100
golden_record_proxy       = C_profiled / T_col * 100      # label Unverified proxy
glossary_coverage         = G / T_gloss * 100
lineage_coverage          = L / T_lineage * 100
```

Guard every division: if a denominator is 0 (or, for classification, both terms' denominators are 0), that measure is **undefined** — report it as `Unverified` with the reason, never as `0%`. Round to 2 decimals; use 4 decimals if the result is below 0.10%. `policy_coverage` is always a whole number.
