# Changelog

All notable changes to the Real Estate Data API are recorded here. The format is
based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

The single source of truth for the deployed version is the [VERSION](VERSION) file,
which is bundled into the Lambda packages and surfaced by `GET /health`.

## [0.5.2] - 2026-10-10

### Changed
- **Tyresö for-sale advertisements refreshed (2026-10-10):** 417 current ads,
  147 new; 168 delisted ads removed.

## [0.5.1] - 2026-10-05

### Fixed
- **First request after Aurora auto-pause no longer returns 500.** The database
  connection now uses a short per-attempt timeout and retries while the cluster
  resumes; if it is still unavailable the API answers `503` with `Retry-After: 10`
  instead of a raw 500 / gateway timeout.

## [0.5.0] - 2026-08-29

### Added
- **Sample size behind every sold price: `sold_obs_{br,villa,radhus,fritids}_<year>`**
  on `cities`, `kommuns` and `areas` (2021–2026). Each column is the number of
  completed sales that fed the paired `sold_kr_per_m2_<type>_<year>` average, so a
  price backed by 2 sales is now distinguishable from one backed by 200. Previously
  only bostadsrätt had a deal count (`num_sales_br_*`), and the villa/radhus/fritids
  price averages carried no denominator at all. The counts come from the existing
  aggregation pass — no new source data and no extra scan of `stg_sales`.

### Known limitations
- **`sold_obs_*` and `num_sales_br_*` count different populations and are not
  interchangeable.** `sold_obs_*` follows the *object_form* axis (physical dwelling
  form) that the price columns use; `num_sales_br_*` follows the *tenure* axis
  (`tenure = 'bostadsrätt'`) that turnover needs. A BRF row-house counts in
  `sold_obs_radhus_*` but in `num_sales_br_*`. Use `sold_obs_<type>` to qualify a
  price and `num_sales_br` only as the turnover numerator; summing `sold_obs_*`
  across types will not equal `num_sales_br_*`.
- The counts are per-year *self* columns only — like the rest of the per-year
  series, they are not copied down onto child rows as `kommun_*` / `city_*`.
- Values are NULL (not 0) for a type-year with no sale, matching the paired price.
- Backfill requires a full reimport against a freshly provisioned schema; the
  columns are empty until then.

## [0.4.5] - 2026-08-23

### Changed
- **Full for-sale (advertisements) refresh across every loaded kommun** —
  Stockholm, Nacka, Solna and Tyresö. The ad set was replaced so sold/withdrawn
  listings drop off: **10,294 → 11,160 advertisements** (2,473 new, 1,607 gone,
  8,687 carried over). Per kommun: Stockholm 7,470 → **8,155**, Nacka 1,131 →
  **1,278**, Solna 1,277 → **1,289**, Tyresö 416 → **438**. Photos re-enriched —
  10,968/11,160 (98.3%) carry `photo_urls`. Sold listings and avgift are
  unchanged (this is a for-sale snapshot only).

### Fixed
- **Ad scrapes now union both capture methods, which miss opposite things.**
  Område-iteration only visits områden already in `booli_areas.csv`, so a short
  driver silently loses whole districts — it returned just 878 Solna ads against
  the true 1,289 (and cost Nacka 230). The kommun-wide SERP covers the whole
  kommun but is paged and stops at `--max-pages`. Both passes now run and are
  merged deduped on `listing_id`: iteration was a strict subset of the SERP on
  all three small kommuns, while Stockholm was the reverse (1,151
  iteration-only vs 23 SERP-only).
- **Corrected a misdiagnosis in `docs/notes/stockholm-circles.md`.** The 3,514
  ads the whole-kommun SERP returned in 0.4.4 were attributed to Booli
  "silently truncating". They were in fact `--max-pages 100` (~35 listings/page)
  — our own cap. Re-run at `--max-pages 200` the same SERP returned 7,004 and
  still reported `page 1/200`. Område-iteration remains the right method for
  Stockholm, but because it covers the kommun more completely, not because the
  SERP truncates.

### Known limitations
- **Solna's sold data is missing Järvastaden, Bergshamra and Vireberg entirely**
  (0 rows each). `sales_booli.csv` was built by område-iteration off the same
  13-område driver that under-covered Solna's ads, so the same districts are
  absent from sold history. Ads for those districts are now correct (recovered
  via the SERP pass); the sold gap needs the områden added via
  `resolve_areas.py` plus a Solna sold re-scrape and enrichment re-run.
- Stockholm's 133-område driver has a smaller residual fringe gap — 23 ads found
  only by the SERP, clustered in Vällingby, Kälvesta, Grimsta and Västberga.

## [0.4.4] - 2026-08-02

### Changed
- **Stockholm expanded from three capture circles to the WHOLE kommun** (areaId 1).
  Added the previously-uncovered NW/W stadsdelar — Bromma, Rinkeby-Kista,
  Spånga-Tensta, Hässelby-Vällingby, Skärholmen, and western Hägersten — by
  extending `booli_areas.csv` with 71 new områden (133 total) and running the full
  pipeline (sold + for-sale + BRF links + BRF details + avgift + photos) over the
  whole kommun:
  - **Sold: 129,818 → 227,664** (191,517 bostadsrätt); coverage now spans the whole
    kommun (lat 59.235–59.418, lon 17.796–18.148) instead of the three circles.
  - **For-sale ads: 4,014 → 7,470** (10,294 total across all kommuns).
  - **BRF details: 4,875 → 6,187** föreningar; 290,524 residences.
  - **Avgift backfilled** for the new scope (`--sold-years 5`, ~30k detail fetches):
    Stockholm avgift **65% → 97%**.
- **`audit_coverage.py` now gates Stockholm as a whole kommun** rather than by
  circle; the three former circles are retained as informational, ungated
  sub-scopes. All scopes pass ≥90% on BRF-link, avgift, and photos.
- **`tenure` now allows `ägarlägenhet`** (owner-apartment — individual freehold of
  an apartment unit, distinct from a bostadsrätt coop share), a fifth property-right
  alongside `bostadsrätt`/`äganderätt`/`tomträtt`/`arrende`. Surfaced by the
  whole-kommun expansion (a single Spånga ad); added to the `advertisements` tenure
  CHECK constraint.

## [0.4.3] - 2026-07-27

### Fixed
- **Southwest Stockholm circle BRF backfill.** The SW circle (v0.4.1) shipped with
  sold+ads+avgift but the BRF enrichment steps (`--enrich-brf`,
  `--load-brf-details`) were skipped, leaving 69% of its bostadsrätt sold rows
  unlinked and every `brf_*` column NULL. Ran both steps: SW BRF-link 31% → 99%,
  adding 262 föreningar (incl. Brf Prästgårdsgränd) and regenerating
  `brf_residences` — all scopes now ≥98% linked.

### Added
- **`src/db/audit_coverage.py`** — enrichment-completeness gate. Reports BRF-link,
  avgift, and photo coverage per kommun / Stockholm circle and exits non-zero if
  any dimension is below threshold. Now a required pre-deploy check (runbook
  Step 6) so a partial Step-3 enrichment chain can't reach a deploy silently.

## [0.4.2] - 2026-07-27

### Changed
- **Full for-sale (advertisements) refresh** across the entire coverage footprint:
  Tyresö, Solna, and Nacka (full kommuns) plus Stockholm's three capture circles
  (central 59.326,18.070,3.4 km; SW 59.275,18.025,3.0 km; Söderort E
  59.269,18.110,4.0 km). Re-scraped every kommun and replaced the ad set so
  sold/withdrawn listings drop off. Now **6,838 current advertisements** (was
  7,106): Tyresö 416, Solna 1,277, Nacka 1,131, Stockholm 4,014. Circle overlap
  deduped on `listing_id` (438 dropped). Photos re-enriched — 6,739/6,838 (98.5%)
  carry `photo_urls`. Sold listings and avgift are unchanged (this is a for-sale
  snapshot only).
- **`tenure` now allows `arrende`** (leasehold plot — building owned, land
  privately leased), a fourth property-right alongside
  `bostadsrätt`/`äganderätt`/`tomträtt`. Six refreshed house ads carry it; the
  `advertisements_tenure_check` constraint was widened to admit them.

## [0.4.1] - 2026-07-27

### Added
- **Southwest Stockholm circle** (3 km radius @ 59.2752, 18.0245): completed
  sales, advertisements, and avgift across the southern Söderort områden (Örby,
  Bandhagen, Stureby, Högdalen, Svedmyra, Enskede, Årsta, Älvsjö, Hagsätra,
  Rågsved, …). Adds 19,312 completed sales and 626 advertisements. Avgift resolved
  for 100% of the last 5 years of bostadsrätt sales — 96.5% carry `monthly_fee_kr`,
  the remaining 3.5% are genuinely fee-less on Booli. Overlap with the existing
  central circle was merged fill-only, preserving prior enrichment.
- **`tenure` now allows `tomträtt`** (site-leasehold — own the building, lease the
  land) alongside `bostadsrätt`/`äganderätt`, a backwards-compatible widening of
  the `advertisements.tenure` check constraint.

## [0.4.0] - 2026-07-20

### Added
- **Central Stockholm circle** (3.4 km radius @ 59.326, 18.070): completed sales,
  advertisements, BRF details/residences, and photos across the inner-city områden
  (Södermalm, Norrmalm, Östermalm, Kungsholmen, Vasastan, Gamla Stan, Djurgården, …).
- **Two-axis property classification**: `object_form` (physical form —
  lägenhet/villa/radhus/parhus/kedjehus/fritidshus) and `tenure` (ownership —
  bostadsrätt/äganderätt) on `sales` and `advertisements`. `property_type` retained
  as a derived, backwards-compatible alias.
- **Avgift (monthly fee) backfill** for the circle — ~96.6% of the last 5 years of
  bostadsrätt sales carry `monthly_fee_kr`; the remainder are genuinely fee-less on
  Booli.

### Fixed
- Sold-detail fee classifier now recognizes Booli's current `SoldProperty` page
  structure: genuinely fee-less pages are classified `no_fee` (definitive) instead
  of `challenged` (retryable), so they are no longer re-fetched indefinitely.

### Known limitations
- `tenure` is authoritative on advertisements but heuristic on sold rows (the sold
  SERP omits `tenureForm`); radhus-in-BRF sold tenure is left NULL.

## [0.3.0] - 2026-07-06

### Added
- `GET /` discovery document — service self-description with the deployed
  version, links to the published documentation (OpenAPI spec, usage guide,
  coverage page, changelog, llms.txt), and the endpoint list.
- `GET /llms.txt` — 302 redirect to the canonical llms.txt on the
  documentation site.

## [0.2.0] - 2026-07-04

### Added
- Two orthogonal property-classification axes, replacing the single conflated
  `property_type` enum (which is kept as a derived, back-compat alias):
  - `object_form` — physical dwelling form (`lägenhet`/`villa`/`radhus`/`parhus`/
    `kedjehus`/`fritidshus`).
  - `tenure` — property-right (`bostadsrätt` / `äganderätt`).
- `GET /advertisements` filters `object_form` and `tenure` (combinable, e.g.
  `?object_form=radhus&tenure=bostadsrätt` for a BRF radhus). Both are validated.
- Both axes are surfaced on advertisement responses.

### Changed
- Per-type rollups now segment by the correct axis: **price/size by `object_form`**,
  **fee/loan/turnover by `tenure`** (bostadsrätt-only). A BRF radhus now folds into
  `_br` economics (previously lost in `_radhus`) while its price stays in `_radhus`.
- Importer `derive_axes` recovers real tenure from on-disk signals (a for-sale
  listing's `attributes_json.tenure_form`, and the BRF link) without re-scraping.
- Stockholm coverage expanded (≈1347 → ≈3519 current listings).

### Fixed
- `flatten_brf_fee_trend`: guard the all-history `regr_slope` against near-zero
  time-variance (near-simultaneous fee-bearing sales), which produced an explosive
  slope that overflowed the `numeric(6,1)` cast — mirrors the existing price-trend
  guard. (Pre-existing bug, exposed by the expanded Stockholm data.)

### Known limitations
- Stockholm/Nacka sold-avgift backfill is partial (≈21%/46% fill), so their fee
  *trends* are noisier until the backfill completes and the schema is re-provisioned.
- Sold `radhus` tenure stays coarse and `parhus`/`kedjehus` fold into `radhus` for
  form until a full re-scrape carries the first-class `object_form`/`tenure` columns.

## [0.1.0]

### Added
- Initial service: completed-sales stats (`/sales/stats/avg-price-per-sqm`),
  property advertisements (`/advertisements`), and the geo/BRF reference endpoints
  (`/kommuns`, `/areas`, `/cities`, `/brfs`), backed by the denormalized
  One-Big-Table read model on Aurora with blue/green schema provisioning.
