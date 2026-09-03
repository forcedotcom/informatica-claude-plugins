# Shared catalog-source topology

Several measures need the same primary-origin list. Compute it once, then reuse it for catalog coverage, freshness, metadata completeness, and golden-record.

## How to build the lists

1. `search_assets` `mode=KEYWORD`, `query=*`, `filterSpec.classType=["core.Resource"]`, `filterSpec.assetLifecycle=["Published"]`, `size=50` (paginate if `total_matches` > size).
2. Each **primary** `core.Resource` is a catalog source. `resourceType` **Reference Catalog Source** is a child — never count it as a separate catalog source.
3. Include **Informatica Data Management Cloud** (`origin` `idmc~catalog`) if it has **Published** assets; it may not appear as a named `core.Resource`.
4. Exclude **CDGC** and **MCC** origins — governance / orchestration, not catalog sources.
5. Count **Published** objects with **batched** `search_assets` (do **not** issue one call per origin when avoidable):
   - Origin list = primary origins from steps 1–4, plus `idmc~catalog` when checking IDMC.
   - Split into batches of **at most 10** origins (aggregation returns max ~10 buckets; larger batches drop smaller origins).
   - Per batch: `query=*`, `mode=KEYWORD`, `size=1`, `filterSpec.origin=[...batch...]`, `filterSpec.assetLifecycle=["Published"]`, `aggregationSpec=[{name:"byOrigin", attributeNames:["core.origin"]}]`.
   - Bucket `key` → origin, `count` → Published object count. An origin in the batch but missing from buckets has count `0`.
   - Do **not** count Draft, Obsolete, or any other non-Published lifecycle.
   - Fallback: if a batch is incomplete, retry that batch (or only the missing origins) with per-origin `size=1` `total_matches`.

Do **not** use a single global Published + origin aggregation for the full catalog — it only returns the top ~10 origins and under-counts `N` / `T`.

## Keep two lists

- **All primary sources** (`T_src`, includes sources with exactly 1 Published object) — total catalog sources, plus IDMC if present. Do not add Reference Catalog Source children.
- **Sources with Published object count > 1** (`N`) — in-scope set for coverage and freshness.

Do not double-count child reference sources that share a parent origin (`snowflake_demo`, `CDI_LOAD_DW`).
