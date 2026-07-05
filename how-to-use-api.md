# Using the Real Estate Data API

## Why this service exists

Buying a home in Sweden is a research problem: comparing areas and price
levels, judging a housing cooperative's (BRF's) financial health, weighing
monthly fees against asking prices, factoring in schools — across thousands of
listings that change daily. An LLM agent is a natural fit for this work, but
only if it has data it can actually reason over.

This service exists to make **your LLM agent amazingly helpful at finding a
property to buy in Sweden**. It is built for LLM consumption first:

- **Answers, not pages.** Endpoints return small, self-contained result sets
  (capped at 10 rows) chosen by *sorting on what you are optimising for* —
  cheapest per m², healthiest BRF finances, slowest fee growth — instead of
  forcing the agent to page through the whole market.
- **Everything needed to reason, in one row.** Each listing embeds its
  cooperative's financials, per-year price and fee history with trends and
  forecasts, and the nearest schools — no follow-up joins, no scraping.
- **Honest about its limits.** Coverage, data gaps and their implications are
  documented (see [coverage](./coverage.md)), so an agent can tell "no results"
  from "not covered" and qualify its advice accordingly.

The intended impact: a person tells their agent what home they are looking
for, and the agent — using this API as its research tool — comes back with
shortlists, comparisons and warnings that would otherwise take weeks of manual
work. The larger ambition is a shift in how the research itself is done: when
the agent-driven path is genuinely better, people spend their search in a
conversation with their LLM rather than clicking through listing pages.

## Usage scenarios

Different buyers come to this research with different demands and mindsets,
and the API supports several distinct strategies. An agent serves its user
best by recognising which of these (often a mix) the user is actually in:

- **Location first.** "Where should we live?" comes before everything else.
  Narrow geographically step by step — kommun, then område within it, then
  the BRFs in that område, and only then compare specific objects in the
  BRFs. The geographic catalogue endpoints (`/kommuns`, `/areas`) and their
  per-area statistics carry the early steps; `/brfs` and `/advertisements`
  the later ones.

- **Best object wins.** Location is flexible; criteria are not. Hunt across
  the whole covered market for the strongest match — best value per m²,
  largest area within budget, lowest monthly cost — by sorting and filtering
  `/advertisements` directly, skipping the geographic funnel entirely.

- **Cooperative quality first.** A bostadsrätt purchase is a stake in the
  BRF's finances, so risk-averse buyers invert the search: find financially
  healthy föreningar first (low debt per m², slow fee growth, stable
  turnover on `/brfs`), then look at what those BRFs currently have for
  sale.

- **Understand the market before searching.** Calibrate expectations first:
  what does a m² cost in each kommun, how have prices and fees moved, what
  can the budget realistically buy where. The per-kommun price statistics
  on `/kommuns` and the sales statistics endpoints answer this without
  touching a single listing.

- **Watch and wait.** The buyer knows what they want and is waiting for it
  to appear. Re-running a saved narrow query sorted by newest listing
  approximates a watchlist until true monitoring exists.

These strategies compose: a typical serious search calibrates on the market,
narrows location, vets cooperatives, and only then compares objects — the
sections below document the endpoints in roughly that order.

## How the API is structured

The API is read-only: plain HTTPS `GET` requests returning JSON, no
authentication. The full contract is the
[OpenAPI spec](https://llm-brf.github.io/openapi.yaml); this section is the
mental model behind it.

The endpoints mirror how the Swedish market is organised — a geographic
hierarchy with the cooperative as the crucial middle layer:

```text
city  →  kommun  →  område  →  BRF (cooperative)  →  individual listing
/cities  /kommuns   /areas      /brfs                 /advertisements
```

- **Catalogue endpoints** (`/cities`, `/kommuns`, `/areas`) enumerate the
  geography present in the dataset, each level with aggregated statistics
  (listing counts, price and fee levels). They answer "what exists and what
  does it cost around here" and supply the exact values the filters on the
  other endpoints accept.
- **Research endpoints** (`/brfs`, `/advertisements`) carry the actual
  comparison work: one row per cooperative or listing, with rich filters
  and sorts.
- **Statistics** (`/sales/stats/avg-price-per-sqm`) aggregate completed
  sales for market calibration, and `/health` reports service status and
  the deployed version.

### Conventions: what is filterable and sortable

Fields that encode a decision axis — something a buyer optimises or
constrains — are exposed for filtering and sorting under two naming
patterns:

- **Numeric axes** (price, size, fees, BRF debt, trends, school indexes) get
  range filters named `min_X` / `max_X` and sorts named `X_asc` / `X_desc` —
  e.g. `max_fee_per_sqm`, `min_brf_units`, `price_per_sqm_asc`.
- **Identity and geography** (kommun, område, BRF) get exact-match filters
  (`kommun`, `omrade`, `brf_org_number`). Exact values are discovered, not
  guessed: the catalogue endpoints enumerate them, and `name_search` /
  `brf_name_search` do case-insensitive substring lookup when only a
  fragment is known.

Descriptive payload fields (addresses, URLs, free text) are returned in rows
but are never filterable or sortable.

The [OpenAPI spec](https://llm-brf.github.io/openapi.yaml) is the complete
inventory: each endpoint's parameter list and its `sort` enum define exactly
what exists. If a parameter is not in the spec, the API does not support it —
do not invent plausible-looking ones.

## Principles of using it

- **Rows are self-contained — no joins.** The data model is deliberately
  denormalized: a listing row embeds its cooperative's financials (per-year
  price and fee history, trends, forecasts) and the nearest schools; a BRF
  row embeds aggregates over its current listings. One request returns
  everything needed to reason about a result.

- **Sort for what you are optimising, then narrow — don't page.** Result
  sets are capped at 10 rows and there is no pagination. This is a feature:
  instead of crawling the market, pick the `sort` that matches the goal
  (best value per m², least-indebted BRF, slowest fee growth, best
  schools…) and tighten filters until the top 10 *are* the answer.

- **Discover values, don't guess them.** Geographic filters match exactly
  (`Tyresö`, not `Tyreso`). Enumerate valid values first: `/kommuns` for
  the `kommun` filter, `/areas` for `omrade` — both exist precisely so that
  filter values never have to be invented.

- **An empty result means "check the premise", not "no such homes".** The
  dataset covers [specific areas](./coverage.md) and some fields are
  backfilled progressively. Before concluding that nothing matches, confirm
  the location is covered and loosen one filter at a time to find which
  constraint bit.

- **Cheap requests, current data.** Catalogue and BRF endpoints are backed
  by small reference tables refreshed on each data import, not live scans —
  iterating on queries is inexpensive, and responses reflect the latest
  import rather than a live crawl.

## Making your first requests

The production base URL is:

```text
https://wqyh106uo0.execute-api.eu-north-1.amazonaws.com
```

No API key, no headers — start with the health check:

```bash
curl "https://wqyh106uo0.execute-api.eu-north-1.amazonaws.com/health"
# {"status": "ok", "version": "0.2.0"}
```

A worked example of the research pattern — "best-value apartments with 3+
rooms in Tyresö":

```bash
curl "https://wqyh106uo0.execute-api.eu-north-1.amazonaws.com/advertisements?kommun=Tyres%C3%B6&object_form=l%C3%A4genhet&min_rooms=3&sort=price_per_sqm_asc"
```

Note the URL-encoding: kommun and område names contain Swedish characters
(`Tyresö` → `Tyres%C3%B6`), and unencoded values will not match. The response
holds the 10 best matches, each row self-contained (~190 fields):

```json
{
  "count": 10,
  "advertisements": [
    {
      "kommun": "Tyresö",
      "asking_price_kr": 1695000,
      "kr_per_m2": 18800,
      "rooms": 4.0,
      "living_area_m2": 90.0,
      "monthly_fee_kr": 7275,
      "brf_name": "HSB BRF Sjötungan",
      "avg_school_socio_index": 136.4,
      "…": "plus the BRF's financials, history and trends, and nearby schools"
    }
  ],
  "filters": { "…": "the filters as applied" }
}
```

## Versioning and change tracking

The service is versioned with semver. `GET /health` reports the deployed
version, and the [changelog](./CHANGELOG.md) documents what changed in each
release — including **Known limitations**, where data-completeness caveats
(like the fee backfill status) are tracked release by release. Coverage
changes (new kommuner) appear both there and on the
[coverage page](./coverage.md).
