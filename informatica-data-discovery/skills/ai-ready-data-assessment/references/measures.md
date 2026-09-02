# Measure query logic

Run these after [topology.md](topology.md). Follow each dedicated skill's grain and call budget. **Empty query results are 0%**, not undefined — see [formulas.md](formulas.md). User inputs `A`, `D`, and `U` are gathered once after query-derived numbers are known — see the parent SKILL.md. **Do not ask for `A_class`.**

`get_asset_details` segment enum is US `neighborhood`. If the schema rejects UK `neighbourhood`, retry `neighborhood`; the payload may still return `neighbourhood`. `size` cannot exceed **100**.

**The class-type trap.** `filterSpec.classType` matches an asset's literal `classType` field, **not** its type hierarchy. `core.DataSet` and `core.DataElement` are abstract types that appear only inside an asset's `types[]` array; a real classified Oracle column has `classType: "…relational.Column"` with `core.DataElement` in `types[]`. Filtering on the abstract values therefore returns a denominator built only from reference-catalog assets and a numerator of **0**. `filterSpec.types` does not rescue this — the server ignores it and returns the entire Published catalog. Any measure that needs data-set / data-element grain must enumerate concrete class types (see measure 6). When a measure's numerator is 0 but its denominator is large, suspect this trap before reporting 0%.

**Transient vs disabled.** NL 403 = NL disabled — but it is intermittent, so retry once before concluding; it is not an empty catalog. NL 502 (`upstream service is unreachable`) and 503 (`no healthy upstream`) are transient outages — retry with backoff of roughly 8 s, 15 s, 30 s. A 503 can take NL *and* KEYWORD down together; a 502 has been NL-only. One cheap KEYWORD `size=1` call tells which. Never read 403/502/503 as a real 0 until the retry ladder is exhausted.

## 1. Catalog coverage

Follow `cdgc-catalog-coverage`. `N` = in-scope sources (Published object count > 1). `A` = additional catalog sources still needing a scan. Coverage = `N / (N + A) * 100`. List the `N` sources sorted by Published object count descending. Sources with exactly 1 Published object are omitted from `N` but still counted in `T_src`. If `N + A = 0`, coverage is **0%**.

## 2. Catalog Freshness-SLA compliance

Follow `cdgc-catalog-freshness-sla`. Same `N` as coverage.

Scan timestamp — use **Last Scanned On** only:

1. `core.lastScannedOn` (date `YYYY-MM-DD`; treat as 00:00:00 UTC) from `get_asset_details` `segments=["summary","selfAttributes"]` on the primary Resource.
2. Else, for Informatica Data Management Cloud only (no Resource): most recent `modifiedOn` among Published `idmc~catalog` assets (`sortSpec` `core.modifiedon` desc, `size=1`).
3. Else: not scanned — do **not** count in `S`.

Do **not** use `core.lastFullScanOn`. Do not use Resource `modifiedOn` when `lastScannedOn` exists. `get_asset_details` is one identity per call — cannot batch.

A source is within SLA when last scan ≥ (now UTC − `D` days), boundary inclusive. `freshness_sla = S / N * 100`. If `N = 0`, freshness is **0%**.

`D` default is **15**. Offer 7, 15, 30, or custom. If the user replies with nothing / “default” / “ok” for `D` only, set `D = 15`.

## 3. Metadata completeness

Follow `cdgc-metadata-completeness-measure`.

- `T_meta` = Published assets on catalog-source origins. One KEYWORD `query=*`, `filterSpec.origin=[all primary origin IDs including idmc~catalog]`, `assetLifecycle=["Published"]`, `size=1`. `T_meta` = `summary.total_matches`. If `originAgg` omits origins, one follow-up KEYWORD with leftover origin IDs only. Do not fan out one call per origin.
- `E` = unique technical identities with at least one of: **business name**, **business description**, **glossary association**, **data classification**, **certification**, **rating**. Do not count inherited Governance Administrator stakeholders. Do not count CDGC-origin glossary/business assets in `E` or `T_meta`.

Keep **separate** NL queries (same origin list, Published, `size=1`): `assets with business name`; `assets with business description`; `show columns related to business term`; `Assets with classifications`. Do **not** OR those strings into one prompt (HTTP 502). Per-origin classified NL undercounts — always batch origins. If classified `total_matches` is at most a few hundred, page `size=100` and add classified-only identities.

Certified: KEYWORD `certified=true`, same origin list. Rated: KEYWORD `averageRating={operator:gt,value:0}`, same origin list. Union identities. Practical union: start from business-name `total_matches`, add classified-only, certified-only, rated-only. Description and glossary are usually inside business name.

If batched NL returns 0 for **both** business name and classifications (or NL 403), `E = 0` and metadata completeness is **0%** (still report `T_meta`). Do not substitute KEYWORD `*` for those NL queries.

## 4. Policy coverage

Follow `cdgc-policy-coverage`. This is **not** a PDP live-path check and is **not** always 100%.

Two `search_assets` calls:

1. KEYWORD `query=*`, `size=1`, Published, `classType=["core.DataElement","core.DataSet","core.DataElementClassification","core.DataEntityClassification","core.DocumentClassification"]`, classType aggregation. `T_pol` = `total_matches` (`T_de` + `T_ds` + `T_class`).
2. NL `query=all objects related to policy`, `size=1`, Published, **no** `classType` filter. `R_pol` = `total_matches`.

Do not put the five `T` classes on the `R` query (related hits may be governance `DataElement` / `DataSet`, not `core.*`). Do not use MCC `MetadataAccessControlPolicy` or governance `Policy` as `T`. Policy objects **do** count in `R` when NL returns them.

If NL returns 0, `R_pol = 0` and policy coverage is **0%** (still report `T_pol`). If NL 403 persists after one retry, same (`R_pol = 0`, **0%**). Sample a non-Policy hit with `get_asset_details` neighbourhood for a policy association when `R_pol` is small enough.

## 5. Unstructured index

Follow `cdgc-unstructured-index`.

`C_unstr` = Published objects whose `classType` is exactly `com.infa.odin.models.unstructuredv2.file.UnstructuredFile`. One KEYWORD `query=*`, `size=1`, Published, that `classType`. Do **not** count by `resourceType` (Amazon S3, SharePoint, …) — that over-counts flat-file metadata. Do not add FlatFile, Folder, Bucket, `core.DataSet`, or `core.Document`.

If `C_unstr = 0`, confirm with **one** extra KEYWORD listing naming variants (`unstructuredv2.file`, `unstructured.file`, `unstructuredv2.UnstructuredFile`, `file.unstructured`, `core.UnstructuredFile`) plus classType aggregation. Catalog-wide classType aggregation absence is **not** evidence of zero.

`U` is user input. `unstructured_index = C_unstr / (C_unstr + U) * 100`. If `C_unstr + U = 0`, report **0%**. If `C_unstr = 0` and `U > 0`, report **0%**.

## 6. Classification measure

Follow `cdgc-classification-measure`. **Not** a blended column/inventory average. **Not** using `A_class`.

`core.DataSet` and `core.DataElement` are **abstract** types that live in an asset's `types[]` array, and `filterSpec.classType` matches only the literal `classType` field. Filtering on them therefore excludes every classified asset and forces `C_cls = 0` against an inflated-looking `T_cls` — see the class-type trap note at the top of this file. Use the concrete 12-class list below for **both** calls; identical lists on numerator and denominator is what makes the ratio meaningful.

```
com.infa.odin.models.relational.Column
com.infa.odin.models.relational.ViewColumn
com.infa.odin.models.relational.Calculation
com.infa.odin.models.relational.ResultSetColumn
com.infa.odin.models.file.flat.FlatField
core.DataElement
com.infa.odin.models.relational.Table
com.infa.odin.models.relational.View
com.infa.odin.models.relational.ResultSet
com.infa.odin.models.relational.ExternalTable
com.infa.odin.models.file.flat.FlatFile
core.DataSet
```

The first six are data elements (`types[]` contains `core.DataElement`), the last six data sets (`types[]` contains `core.DataSet`). Exclude `PrimaryKey`, `ForeignKey`, `Tag`, `Schema`, `Parameter`, `Statement`, `ProcedureDefinition`, `Folder`, `Bucket`, `core.Resource` — neither grain.

Two `search_assets` calls:

1. KEYWORD `query=*`, `size=1`, Published, `classType=` the 12-class list, classType aggregation. `T_cls` = `total_matches`. Sum the element classes for `T_de` and the set classes for `T_ds`.
2. NL `query=data sets and data elements related to classification`, `size=1`, Published, `classType=` the **same** 12-class list, **no** `aggregationSpec` (NL never returns aggregations). `C_cls` = `total_matches`.

The aggregation caps at ~10 buckets, so with 12 classes the buckets will **not** sum to `total_matches`. That is the cap, not leakage — attribute the remainder to the small classes that fell outside the top 10 (typically `ResultSet` and `ExternalTable`).

`filterSpec.types` is **ignored** by the server — it returns the whole Published catalog. Never use it to select by grain.

**Before accepting `C_cls = 0`**, re-run the same NL query with **no** `classType` filter. If that total is higher than `C_cls`, the class list is missing a concrete class present on this tenant (a new scanner type, e.g. `unstructuredv2.file.*`) — identify it by paging `size=100` and grouping on `classType`, add it to both calls, and recompute. Only report **0%** when the unfiltered total is also 0. Add a new class only after confirming `core.DataElement` or `core.DataSet` appears in a hit's `types[]`.

Do not query MCC classification catalogs, "classifications related to any objects", or additional-classifications-to-add. A classification's `neighbourhood` returns only glossary associations — it never lists the assets it classifies, so `C_cls` cannot be built by enumerating classifications.

Sample `get_asset_details` `segments=["summary","dataClassification"]` on a handful of hits. Unlike lineage, this check **works**: a classified asset returns entries with `core.name` and `core.curationStatus` (e.g. `ACCEPTED`). If several samples come back with an empty `dataClassification`, treat `C_cls` as untrustworthy and say so in limitations.

If NL returns 0 with the full class list and the unfiltered check also returns 0 (or NL 403 persists after retry, or 502/503 after the backoff ladder), `C_cls = 0` and classification measure is **0%** (still report `T_cls`). Do not substitute KEYWORD `*`.

## 7. Data-quality score

Follow `cdgc-data-quality-score`. Grain: unique **technical objects** (columns), not RuleInstance records.

- `D` = unique primary parents (segment above the RI identity in `location` / `path`) plus secondary `toIdentity` where `associationKind` is `com.infa.ccgf.models.governance.secondaryObject`.
- `Q` = unique identities in `D` with at least one Completeness, Validity, or Accuracy rule.

**Call reduction:** do **not** `get_asset_details` every RuleInstance. Parse primaries from search hits. Infer dimension from search `name` + `description` first. Details (`selfAttributes`) only if still unlabeled. Neighbourhood: probe up to **30** RIs; if zero secondaries, skip the rest and report secondaries = 0; if any secondary, fetch neighbourhood for remaining RIs.

Dimension inference: Completeness = null/not-null/missing/empty; Validity = format/domain/allowed-values; Accuracy = correct vs reference (including ISO country-name mapping). Unlabeled rules stay in `D` but do not lift `Q` unless another attached rule qualifies. Do not treat Consistency as Completeness/Validity/Accuracy.

If zero RuleInstances, `D = 0`, `Q = 0`, and the score is **0%**.

## 8. Golden-record match rate

Follow `cdgc-golden-record-match-rate`.

- `T_col` = Published `com.infa.odin.models.relational.Column` `total_matches`. Do not add ViewColumn, ResultSetColumn, or Reference Catalog Source `core.DataElement` to `T_col`.
- `C_profiled` = table columns **plus** view columns whose search document has `core.numValuesProfiled` (including **0**). Missing the attribute is not 0. `get_asset_details` does not return this field — aggregation only.

Per origin (from topology), KEYWORD Column + `aggregationSpec` on `core.numValuesProfiled`:

- Empty buckets → 0 for that origin’s table columns.
- Bucket sum equals `total_matches` → add `total_matches`.
- Buckets non-empty but sum < `total_matches` (10-bucket cap) → remaining columns still have the attribute — add **`total_matches`** for that origin, not the bucket sum.

Repeat for `ViewColumn`. `C_profiled` = table contribution + view contribution.

This is profiling-attribute coverage, not MDM match accuracy — say that in limitations. Label `Observed` when the counts succeeded. If `T_col = 0` (query returned no columns), golden-record is **0%**.

## 9. Glossary coverage

Follow `cdgc-glossary-coverage`.

**Published only** — do not include Draft. One KEYWORD paged query: `query=*`, `size=100`, `classType=["com.infa.ccgf.models.governance.BusinessTerm","com.infa.ccgf.models.governance.Metric"]`, `assetLifecycle=["Published"]`, `sortSpec` `core.name` asc. First page `total_matches` is `T_gloss`. Paginate `from=100, 200, …` until unique identities equal `T`. After the first page, remaining pages in parallel.

`G` from the **same hits**: HTML-stripped description non-empty, or `certified === true`, or non-empty `stakeholders`. Do not issue NL description/stakeholder queries, certified-only filters, or per-term `get_asset_details`. Do not count inherited Governance Administrator lists on technical assets.

If `T_gloss = 0`, glossary coverage is **0%**.

## 10. Lineage coverage

Follow `cdgc-lineage-coverage`.

- `T_lineage` = Published `core.DataSet` + `core.DataElement`. One KEYWORD `query=*`, `size=1`, both classTypes, classType aggregation. Do not add Table, View, Column, Calculation, Statement, Parameter, or AIModel.
- `L` = one NL `query=show assets that participate in lineage`, `size=1`, same lifecycle and classTypes, **no** `aggregationSpec` (NL never returns aggregations).

Do **not** neighbourhood-check `L` (neighbourhood does not expose lineage). Do **not** use an unfiltered NL lineage search (~30,000 cap). Do not use KEYWORD `"lineage"` as `L`.

**502** = transient; retry ~7 s, 15 s, 20 s. Do not treat 502 as `L = 0` until those retries are exhausted; then `L = 0` and lineage is **0%**. **403** = NL disabled; retry once, then `L = 0` and lineage is **0%** (still report `T_lineage`). If NL returns 0 for both classes, `L = 0` and lineage is **0%**.
