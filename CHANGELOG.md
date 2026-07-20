# Changelog

All notable changes to the Real Estate Data API are recorded here. The format is
based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

The single source of truth for the deployed version is the [VERSION](VERSION) file,
which is bundled into the Lambda packages and surfaced by `GET /health`.

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
