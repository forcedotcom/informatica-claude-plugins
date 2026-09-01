# Shared catalog-source topology

Several measures need the same primary-origin list. Compute it once, then reuse it.

## How to build the lists

1. `search_assets` `mode=KEYWORD`, `query=*`, `filterSpec.classType=["core.Resource"]`, `size=50` (paginate if `total_matches` > size).
2. Each **primary** `core.Resource` is a catalog source. `resourceType` **Reference Catalog Source** is a *child* of a primary source — never count it as a separate catalog source (it still counts as its own dataset/object grain inside measures that ask for that).
3. Include **Informatica Data Management Cloud** (`origin` `idmc~catalog`) if it has assets, even though it may not appear as a named `core.Resource`.
4. Exclude **CDGC** and **MCC** origins — governance/orchestration, not catalog sources.
5. For each remaining primary origin, `search_assets` `query=*`, `filterSpec.origin=[origin]`, `size=1`; object count = `summary.total_matches`.

## Keep two lists

- **All primary sources** (`T`, includes sources with exactly 1 object) — denominator for total catalog sources.
- **Sources with object count > 1** (`N`) — the in-scope set for coverage and freshness.

Aggregations cap at ~10 buckets and `aggregationSpec` supports 1 item — never rely on origin aggregation alone for a full list; page through `core.Resource` results instead. Do not double-count child reference sources sharing a parent origin.
