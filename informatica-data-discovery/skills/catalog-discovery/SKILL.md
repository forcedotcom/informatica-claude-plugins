---
name: catalog-discovery
description: "Discover data assets in the CDGC catalog and assemble tenant-specific context (business glossaries, data classifications, stakeholders, ratings, certification) BEFORE reading any actual data through other MCPs. Use this skill whenever the user asks a data question — 'find', 'where is', 'show me', 'what tables have', 'is there a report on', 'who owns', 'what does column X mean', 'give me customer data', etc. Applies to both business assets (domains, glossaries, policies) and technical assets (tables, columns, files, dashboards)."
version: 1.3.0
---

# Catalog Discovery

The CDGC catalog is the authoritative index of what data exists in this tenant, what it means to *this* organization, who owns it, and how trustworthy it is. **Always start here.** Only after the catalog has narrowed the answer to a specific asset (or short list) should another MCP be used to read the underlying data.

## Core Principles

1. **Catalog-first, data-second.** Never query a database, warehouse, S3, or BI MCP directly from a cold user prompt. The catalog tells you which asset is the right one, whether it is certified, who the steward is, and what its tenant-specific meaning is. Reading raw data before doing that risks pulling the wrong table, an uncertified copy, or data the user is not entitled to interpret.
2. **Prefer tenant context over generic knowledge.** If a business glossary term, classification, or stakeholder note exists in *this* catalog, that definition wins over any generic definition the model knows. "Customer", "Revenue", "PII", "Active Account", "last quarter" all mean whatever the tenant's stewards say they mean.
3. **Trust signals matter — but only datasets carry certification.** A `certified: true` asset has been vetted by stewards. `averageRating` runs 1–5; higher is better. When multiple candidates match, rank by certification first, then rating, then recency (`modifiedOn`).

   **Certification lands only on dataset-grain assets** — tables, views, files, BI datasets and reports, SaaS objects. Business terms, domains, policies, columns and other `DataElement`s are **never** certified, whatever their quality. So `certified: false` on a glossary term is not a defect and not a reason to distrust it; for those assets the trust signal is `assetLifecycle` (`Published` > `Draft`) plus the stakeholder who authored it. Never downgrade a term, or omit it from your answer, because it is uncertified.
4. **Read your own aggregations before querying again.** Every search returns `resourceAgg` / `byType` / `originAgg` buckets. These are not decoration — they are the routing table for your next call. Issuing another broad search without reading them is the single most common way to waste a turn.
5. **Session context is a hint, not a filter.** A connected device's name, an already-loaded connector, or last turn's topic is a reason to *also* look somewhere — never a reason to filter to *only* there. Let the aggregations, certification and rating tell you which system holds the trustworthy asset.

## The Two Discovery Tools

| Tool | Purpose |
|---|---|
| `search_assets` | Find assets by keyword, with structured filters and aggregations. **Bounded and cheap** (`size` caps the response) but **ranked and partial** — a better-named asset can be buried below the fold. |
| `get_asset_details` | Fetch full detail for one asset by identity, with selectable segments: `summary`, `selfAttributes`, `stakeholdership`, `dataClassification`, `glossary`, `hierarchy`, `neighbourhood`. **Complete and unranked** for what it returns, but **unbounded** — no `size` parameter. |

Understanding that trade-off is most of the skill. Search chooses a neighbourhood; `hierarchy` exhausts one.

### Mode selection

**Use KEYWORD. Effectively always.** It supports `filterSpec`, `aggregationSpec` and `sortSpec`, and it is predictable.

**NL mode is a last resort**, for a genuine zero-result retry on a conceptual question. It is unreliable in practice: it drifts semantically (a query about revenue *regions* can return weather-forecast tables matching on "forecast"/"region"), and **it fails outright on structural queries** — "show columns from table X", "what is the parent of Y" can return a server error rather than results. Structural traversal is `get_asset_details` + `hierarchy`, never NL.

## Asset Taxonomy (quick reference)

- **Glossary** — `Domain`, `SubDomain`, `BusinessTerm`, `Metric`. The tenant's business vocabulary.
- **Data Catalog (Technical)** — `Resource → DataSource (DB/Schema) → Dataset (Table/View) → DataElement (Column)`, plus files, procedures, keys. Filter with `resourceType` (Oracle, Snowflake, Postgres, …).
- **System & Data Set** — business-friendly wrappers over technical infrastructure.
- **Process, Policy, Project** — governance context.
- **AI** — `AIModel`, `AISystem`.
- **Business Area, Legal Entity, Geography, Regulation** — organizational scope.

---

## Discovery Workflow

### Step 1 — Decompose, and include the compound phrase

Break the request into probes. Each noun that could be a business concept, a physical dataset, an owner, a system, or a policy is a candidate probe.

**Critically: also probe the compound phrase, not just its parts.** Tenant assets are frequently named after the *combined* concept. Searching "customer" and "revenue" as separate glossary probes will not surface a view named `V_CUSTOMER_COMMERCE_METRICS`. Run at least one search on the compound phrase (or close synonyms — "commerce metrics", "revenue per customer") against the whole catalog with no `classType` restriction.

**Identify discriminating vs. dimensional terms.** In "customer revenue for EMEA last quarter", *revenue* and *customer* discriminate; *EMEA* and *last quarter* are dimensions of the answer — they will be a column value and a date filter, not an asset name. Searching hard on a dimensional term burns calls. Probe it once, cheaply, then move on.

### Step 2 — Open with a trust census

Run **one** call to size the trusted set, and aggregate on `classType` while you are there:

```json
{ "query": "*", "mode": "KEYWORD", "filterSpec": { "certified": true }, "size": 1,
  "aggregationSpec": [{ "name": "agg", "attributeNames": ["core.classType"] }] }
```

Read `total_matches`. That number tells you how to weight certification for the rest of the search:

- **Small (tens)** — certification is a strong, usable filter. Apply `certified: true` alongside your real query terms.
- **Large (hundreds+)** — certification is near-universal here and discriminates weakly; lean on rating and recency instead.

Then read the `classType` buckets, and expect them to contain **datasets only** — `Table`, `View`, `PDFFile`, `PowerBI.Cloud.Dataset`, `Salesforce.Object`, `HierarchicalFile` and the like. That is the norm, not a quirk of one tenant: certification is a dataset-grain signal. If a `BusinessTerm`, `Domain` or `Column` ever appears in those buckets, that tenant is doing something unusual and the rest of this section's advice needs rechecking.

The practical consequence: **`certified: true` is a dataset filter in disguise.** Adding it to a probe silently drops every glossary term, policy and column from the result set — so use it on technical-asset searches (Step 3) and never on a glossary probe (Step 6).

> **Never run bare `query: "*"` with `certified: true` and read the results.** With no query term the ranking is arbitrary and the result set is paginated — the asset you want can sit outside the first page of an otherwise tiny set. Pair the certification filter **with** your search terms, always.

### Step 3 — Search technical assets, then read the aggregation

Run the compound-phrase and discriminating-term searches with `aggregationSpec` on `core.resourceType` and `core.classType`, and **no** `resourceType` filter on the first pass.

Then stop and read the buckets. The heuristic that matters:

- **Small buckets under descriptively-named resources are signal.** A resource named for its purpose (`snowflake_sales_customer`, `finance_dw_curated`) holding a handful of matches is a curated, modelled neighbourhood.
- **Large buckets under generic names are noise.** `amazon redshift` with 48 hits, `workday_jdbc` with 33, is almost always a bulk-scanned staging area — dozens of near-identical `*_proxy`, `*_stg`, `*_uat` variants plus foreign keys and constraints.

**Do not chase the biggest bucket.** Volume tracks how much was scanned, not how much was curated.

Useful `filterSpec` shortcuts:

- `"certified": true` — pair with query terms, per Step 2. Technical-asset searches only: it excludes glossary terms and columns by construction.
- `"averageRating": {"operator": "gt", "value": 3}`
- `"assetLifecycle": ["Published"]` — exclude drafts and obsolete. This is the trust filter that works on *every* class, certified or not.
- `"classType": ["com.infa.odin.models.relational.Table"]` — datasets only, excluding columns/keys.
- `"resourceType": ["Snowflake"]` — **only after** the unscoped pass has shown you the spread.

### Step 4 — Check grain before you commit to a candidate

**Do this from the column-name list, before fetching full details.** It is the cheapest disqualifier available.

If the question is scoped to a period ("last quarter", "in March", "year on year"), an asset whose measures are all lifetime or rolling-window cannot answer it:

- `TOTAL_*`, `LIFETIME_*`, `*_TO_DATE` → cumulative, no period cut.
- `L7D_*`, `L30D_*`, `L90D_*` → rolling windows relative to `CURRENT_DATE()`. **`L90D` is not "last quarter."** It is the trailing 90 days ending today. Column descriptions sometimes claim these "support quarterly reviews" — read the definition, not the marketing.
- A `DATE_KEY` / `DATE_VALUE` / `*_DATE` grain column → sums to any calendar or fiscal period. This is what a period-scoped question needs.

For views and aggregates, read `core.sourceStatementText` in `selfAttributes` when present. It gives you the actual revenue/metric definition — status filters, currency conversion, join path — which is what you must reproduce if you end up querying source tables directly.

### Step 5 — Navigate with `hierarchy` (optional, cost-aware)

`get_asset_details(segments: ["hierarchy"])` returns an asset's **children** — complete and unranked. Two uses:

**Down, from a known asset.** Columns of a table or view in one call. Cheap, bounded (typically 10–30 rows), and often decisive — the column list alone settles the grain question in Step 4. Always prefer this over an NL "show columns from X".

**Sideways, for context.** Resolve the parent schema, then read its child list to see what else lives in the neighbourhood — a daily aggregate, the stored procedure that builds it, an ETL audit table carrying freshness. Keyword ranking can bury these entirely; a sibling list cannot.

The sideways walk is **a deliberate choice, not a default.** Before doing it, weigh:

- **Fan-out is unbounded and invisible until the response lands.** `hierarchy` has no `size` or pagination. A curated schema returns a dozen children; a staging schema returns hundreds.
- **The response mixes in non-datasets** — primary keys, foreign keys, procedures, constraints — alongside the tables you want. On a bulk-scanned schema that is the bulk of the payload.
- **Walking *up* costs an extra lookup.** `hierarchy` returns children only, so reaching a parent means resolving it first (from the `path` string, via a targeted search on the database or schema name).

**Cheap signal for whether to bother — reuse the aggregation you already have.** A small bucket under a descriptively-named resource is worth the walk. A large bucket under a generic one is not: stay in search and keep filtering.

### Step 6 — Resolve definition-sensitive language

Before reporting any period-, status-, or segment-scoped number, probe the glossary **for that specific word**. This is a targeted probe, not the broad vocabulary sweep in older versions of this skill.

Trigger it when the question contains:

- **A relative period** — "last quarter", "YTD", "this month", "last week". Fiscal calendars routinely differ from calendar ones. A tenant whose fiscal year starts in February makes "last quarter" May–July, not April–June — a completely different answer with no error message to warn you.
- **A status or segment word** — "active", "churned", "closed-won", "at risk".
- **A metric name with a house definition** — "revenue", "margin", "ARR", "NRR".

```json
{ "query": "fiscal quarter", "mode": "KEYWORD",
  "filterSpec": { "classType": ["com.infa.ccgf.models.governance.BusinessTerm"] } }
```

Prefer `Published` terms over `Draft`, and note the domain when several compete. Quote the steward's definition in your answer and state the concrete date range you derived from it.

**Never add `certified: true` to a glossary probe.** Terms are never certified (Core Principle 3), so the filter returns nothing and you will wrongly conclude the tenant has no definition for the word. `Published` is the trust signal here: a `Published`, uncertified `Fiscal Quarter` term is authoritative and should be quoted as such.

> **Glossary probing is triggered, not blocking.** Older guidance ran a full vocabulary sweep before touching technical assets. In tenants with large or messy glossaries — dozens of near-duplicate `Customer` and `Revenue` terms across demo verticals, mostly `Draft` — that sweep costs several calls and returns nothing usable. Probe the glossary when a term will change the answer (above), or when a candidate asset carries a `glossary` link worth resolving.

### Step 7 — Enrich the finalists

For each serious candidate, `get_asset_details` with the segments you actually need:

- **`glossary`** — the business terms stewards linked to this asset; the tenant meaning of the table.
- **`dataClassification`** — PII/PCI/Confidential and similar. Tells you sensitivity *before* you read data. A `Person` classification on a customer table means the result set carries personal data — say so.
- **`stakeholdership`** — owner / steward / SME. Cite them when trust or interpretation matters; defer to them when confidence is low.
- **`hierarchy`** — columns, per Step 5.
- **`selfAttributes`** — `sourceStatementText`, `technicalDescription`, `NumberOfRows`, certification metadata.
- **`neighbourhood`** — connected assets, lineage, linked reports.

Request only what the question needs. Fetch all segments only when the user explicitly asks for complete details.

**Sanity-check the finalist before recommending it.** `NumberOfRows: 0` means empty *or* never profiled — flag it rather than assuming the table is populated. An uncertified asset with a better grain fit may still beat a certified one with the wrong grain: present both and say why.

### Step 8 — Correlate and answer

- Which assets are both trustworthy *and* actually answer the question at the right grain? Those are your primary answer.
- Do classifications imply access or handling constraints the user should know about?
- Do candidates disagree (different domains, different definitions, different stewards)? Say so; don't paper over it.

Report: asset name, class, resourceType, certification, rating, linked terms with tenant definitions, classifications, steward, and the identity so the user can drill in. State explicitly any gap between what was asked and what the asset provides.

### Step 9 — Hand off to a data-reading MCP

Once discovery has produced a specific identity, pass the resolved path (from `hierarchy` / `path`) into the appropriate MCP — Snowflake, Postgres, S3, BI — to read the actual data.

If the relevant connector is **installed but not enabled in this chat**, its tools will not be in your tool list. Say so plainly and tell the user to enable it in the chat's connector settings, rather than reporting the question as unanswerable.

---

## Handling Ambiguity and Empty Results

- **KEYWORD returned nothing** — broaden the terms, drop `classType`, then and only then try NL.
- **Hundreds of matches** — you are searching a dimensional term, or a staging area. Re-read the aggregation and narrow by resource.
- **Multiple business terms match** — list them with domain and stakeholder; ask, or answer for all with clear labels.
- **No certified asset exists** — say so explicitly, offer the best-rated candidate marked as uncertified, and surface the steward.
- **A `certified: true` probe came back empty** — check what class you were looking for before reporting a gap. On glossary terms, domains, policies or columns the filter can only return zero, because those classes are never certified. Re-run with `assetLifecycle: ["Published"]` instead.
- **No glossary term for a concept** — do not silently substitute a generic definition. Say the catalog has none and label any fallback as your own.
- **A `resourceType` filter came from context rather than a search** — stop, run the unscoped pass first.

## Cheatsheet

- KEYWORD always; NL only on genuine empty results; never NL for structural queries.
- Read `resourceAgg` before the next call. Small named bucket = signal; big generic bucket = staging noise.
- Size the certified set once, then pair `certified: true` **with** query terms — never bare `*`.
- Only datasets are certified. `certified: true` is a dataset filter in disguise — never put it on a glossary probe; a `Published` uncertified term is authoritative.
- Search the compound phrase, not only its parts.
- Separate discriminating terms from dimensional ones; don't hunt for the dimension.
- Grain first: `TOTAL_`/`LIFETIME_` = cumulative, `L30D_`/`L90D_` = rolling (not "last quarter"), `DATE_KEY` = period-capable.
- `hierarchy` down for columns is cheap; `hierarchy` sideways for siblings is a deliberate, cost-checked move.
- Probe the glossary for period/status/metric words that change the answer — not as a blanket opening sweep.
- `NumberOfRows: 0` means empty or unprofiled. Flag it.
- Read `sourceStatementText` before trusting an aggregate or view.
- Discovery ends with an identity; data reading starts with it. Do not skip the handoff.

## Worked illustration of the method

A period-scoped question over a messy multi-vertical tenant, showing the failure mode and the corrected path.

**Wasteful:** four opening searches — two broad glossary sweeps (`revenue`, `customer`) returning ~120 mostly-`Draft` demo terms, plus a hunt on the dimensional term (`EMEA`, 259 matches of staging columns). The aggregations from those calls already named the curated resource; they went unread while further broad searches were fired. NL retries drifted and errored. The winning asset was reached on call ~13.

**Corrected:** size the certified set (1 call) → compound-phrase search with certification and aggregations, read the buckets, spot the small purpose-named resource (1 call) → `get_asset_details` on the top hit with `selfAttributes` + `hierarchy`, which yields the column list *and* the view SQL, settling grain immediately (1 call) → glossary probe on "fiscal quarter" because the question is period-scoped (1 call) → optional sibling walk of the schema, justified because the bucket was small, revealing a daily-grain aggregate and its build procedure that keyword ranking had buried (2 calls).

The general shape: **search to choose a neighbourhood, aggregations to confirm it, `hierarchy` to exhaust it, glossary to pin the definitions, then hand off.**
