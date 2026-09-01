# Measure query logic

Run these after [topology.md](topology.md). User inputs `A`, `D`, `U`, and `A_class` are gathered once after every query-derived number is known — see the parent SKILL.md.

`get_asset_details` segment enum is US `neighborhood`. If the schema rejects UK `neighbourhood`, retry `neighborhood`; the payload may still return `neighbourhood`.

## 1. Catalog coverage

Numerator/denominator: `N` (in-scope sources) over `N + A`, where `A` is additional catalog sources still needing a scan. List the `N` sources sorted by object count descending. Sources with exactly 1 object are excluded from `N` but still counted in `T`.

## 2. Catalog Freshness-SLA compliance

In scope: the same `N` sources. For each, get last-scan time from the primary `core.Resource` via `get_asset_details` `segments=["summary","selfAttributes"]`:

1. `core.lastFullScanOn` (epoch ms, convert to UTC) — preferred.
2. Else `core.lastScannedOn` (date `YYYY-MM-DD`, treat as 00:00:00 UTC).
3. Else, for Informatica Data Management Cloud only (no Resource): most recent `modifiedOn` among `idmc~catalog` assets (`sortSpec` on `core.modifiedon` desc, `size=1`).
4. Else: not scanned — excluded from the "scanned within SLA" count.

Never use `core.modifiedOn` on the Resource when `lastFullScanOn` / `lastScannedOn` exist (it moves on certification/edits unrelated to scanning). A source is "within SLA" when last scan ≥ (now UTC − SLA-window-days), boundary inclusive. Count how many of the `N` sources qualify (`S`); the SLA window in days (`D`) is a user input.

## 3. Metadata completeness

Denominator `T` = every asset on catalog-source origins (`core.IClassTechnical`): sum `summary.total_matches` per primary origin from topology (skip Reference Catalog Source children as extra origins; exclude CDGC/MCC).

Numerator `E` = unique technical objects with at least one of: business name, glossary association, data classification, certification, rating. Count each criterion per origin, then **union identities** (an object counts once even with multiple enrichments):

1. Business name — NL `assets with business name`, per origin, `size=1`.
2. Glossary-linked columns — NL `show columns related to business term`, per origin — usually a subset of (1).
3. Classified assets — NL `Assets with classifications`, per origin — add identities not already counted.
4. Certified (catalog-wide KEYWORD `filterSpec.certified=true`) — dataset-grain; add datasets not already counted.
5. Rated (catalog-wide KEYWORD `filterSpec.averageRating={operator:gt,value:0}`) — drop CDGC-origin hits; add remaining technical identities not already counted.

Do not count inherited Governance Administrator stakeholders as an enrichment. Sample `get_asset_details` `segments=["summary","selfAttributes","glossary","dataClassification"]` to confirm overlap when the union is ambiguous. If NL returns 0 everywhere for business name and classifications, this measure cannot be counted — say so and stop rather than guessing.

## 4. Policy coverage

This measure is a live-path check, not an object count. Agent search always routes through `ccgf-searchv2`, which calls `ccgf-policy-decision-point` to verify the caller has the required Metadata Access Control policy before returning results. So: call `search_assets` with any cheap query (`mode=KEYWORD`, `query=*`, `size=1`).

- Success (a `summary` payload returns, including `total_matches = 0`) → the PDP path is live and every defined access policy in force for agent search is being enforced → **100%**, always a whole number.
- Auth/session failure (`AUTH_01`) → ask for a new `IDS-SESSION-ID`, stop.
- Non-auth outage → say policy coverage cannot be confirmed live right now; do not invent a number.

Never enumerate `core.accesscontrol.MetadataAccessControlPolicy` via `search_assets` (catalog search doesn't index those MCC objects — `0` there does not mean PDP isn't enforcing). Never substitute governance `com.infa.ccgf.models.governance.Policy` (Data Standards/privacy policies). This measure proves the enforcement *path* is live, not how many discrete policies exist or that each is correctly scoped — say that plainly in the report's limitations.

## 5. Unstructured index

Numerator/denominator base `C` = every catalog object (Resource, Bucket, Folder, File/FlatFile, FlatField, etc. — not just datasets) whose `resourceType` is one of: Amazon S3, Google Cloud Storage, Oracle Cloud Object Storage, File System, SFTP File System, Hadoop Distributed File System (HDFS), Microsoft OneDrive, Microsoft SharePoint Online.

`search_assets` `mode=KEYWORD`, `query=*`, `size=1`, once combined across all eight `resourceType` values and once per type (so a true zero is confirmed, not just absent from a capped aggregation). Exclude MCC. `U` (unstructured assets known but not yet in CDGC) is a user input; the ratio is `C / (C + U)`.

## 6. Classification measure

This one blends two sub-ratios (column-level and inventory-level), then averages them — the average itself is an internal computation, not shown in the report as a formula.

- `Tc` = `com.infa.odin.models.relational.Column` count (`filterSpec.classType`, `size=1`).
- `Cc` = columns with at least one `dataClassification` — NL `columns with classifications` on the same classType. Confirm on a sample via `get_asset_details` `segments=["summary","dataClassification"]`.
- `K` = sum of `summary.total_matches` for `core.DataElementClassification`, `core.DataEntityClassification`, `core.DocumentClassification` (three separate KEYWORD calls).
- `R` = classifications assigned to at least one object — NL `classifications related to any objects`, `size=50` paginated; fall back to a union of classified-column identities plus non-column NL hits if the direct query is empty.
- `A_class` (additional classifications still to add) — user input.

Exclude origin-`catalog` generated classes (`externalId` `CL_GEN_*`) from `K`/`R`. `filterSpec.types` is unreliable — always filter on `classType` explicitly. If NL returns 0 for classified columns, that sub-ratio cannot be counted — say so rather than reporting a fabricated column rate.

## 7. Data-quality score

Grain: unique technical objects (columns), not RuleInstance records.

- `D` = unique identities linked to at least one `com.infa.ccgf.models.governance.RuleInstance`, as primary (parent column in the RuleInstance path) or secondary (`neighbourhood` `associationKind = ...secondaryObject`). Enumerate RuleInstances via KEYWORD `classType=["com.infa.ccgf.models.governance.RuleInstance"]`, then `get_asset_details` `segments=["summary","selfAttributes","neighbourhood"]` per instance (retry `neighborhood` spelling if the schema rejects UK spelling).
- `Q` = the subset of `D` where at least one attached rule is Completeness, Validity, or Accuracy — classify via an explicit dimension field if present, else infer from the rule's technical description (null/not-null → Completeness; format/domain/allowed-values → Validity; correct-vs-reference → Accuracy), else unlabeled (excluded from `Q` unless another attached rule qualifies).

If there are zero RuleInstances, `D = 0` and the score is undefined — never invent DQ objects from name matches on "Completeness"/"Validity"/etc.

## 8. Golden-record match rate

**Caution — name vs. what CDGC actually exposes.** What this query measures is *profiling-attribute coverage* (`core.numValuesProfiled` presence on columns), not entity-resolution accuracy, match precision/recall, or survivorship correctness. Compute it because it is a real CDGC signal, but label it explicitly in the report as an **Unverified proxy** for true golden-record quality, and list what a real golden-record accuracy claim would require: a labeled truth set, declared match rules/model version, coverage, and independent validation. Never present this number as if it answers "are records correctly matched."

- `T` = `com.infa.odin.models.relational.Column` count (`size=1`).
- `C` = table + view columns whose search document carries `core.numValuesProfiled` (including the value `0` — missing the attribute entirely is different from present-with-zero). Use per-origin `aggregationSpec` `attributeNames=["core.numValuesProfiled"]` against the topology origin list (10-bucket cap applies — if bucket sum equals `total_matches` for an origin, the whole origin has the attribute). Repeat for `ViewColumn`. `get_asset_details.selfAttributes` does not surface this field — aggregation only.

## 9. Glossary coverage

- `T` = `total_matches` for `com.infa.ccgf.models.governance.BusinessTerm` plus `com.infa.ccgf.models.governance.Metric` (Published + Draft).
- `G` = unique identities with at least one of: non-empty description (HTML-stripped), `certified = true`, or a non-empty stakeholder list. Union across: NL `business terms with description` / `metrics with description` (page to `total_matches`), KEYWORD `filterSpec.certified=true`, NL `show business terms and metrics with stakeholders`.

Certification is usually dataset-grain, so expect 0 certified terms in most tenants — that's expected, not a bug. Don't count inherited technical-asset stakeholder lists as glossary stakeholders. If NL description search returns 0 for both classes, this measure cannot be counted.

## 10. Lineage coverage

- `T` = technical datasets (`Table`/`View`/`ResultSet`/`ExternalTable` on primary `resourceType`s) plus reference datasets (`core.DataSet` on `resourceType=Reference Catalog Source`).
- `L` = the subset of `T` that participates in lineage — NL `show assets that participate in lineage` per class type (KEYWORD can't filter on lineage relationship). Confirm a small sample via `get_asset_details` `segments=["summary","neighbourhood"]` (retry `neighborhood` spelling as needed) — NL can false-positive on the literal word "lineage" in a name; discard samples with an empty neighbourhood.

Exclude CDGC-origin governance `DataSet`/`System` wrappers from both `T` and `L` — they were never scanned. If every NL lineage count is 0, this measure cannot be counted.
