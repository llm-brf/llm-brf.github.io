# Hemklok

**An AI buyer's agent for the Swedish residential property market.**

Tell it your life — dog, kids, commute, budget, noise tolerance — and it
recommends the right areas, flags BRF financial risks, and gives you real
market prices.

### [→ Try the beta](https://chatgpt.com/g/g-6a2980f7c8a88191905fbacd4dfc301e-klokhem) · [hemklok.se docs](https://llm-brf.github.io)

This repository is the public site and API reference. It holds the OpenAPI
spec, the `llms.txt` pair, and an honest account of what the data covers.

## Why it exists

Buying a home in Sweden is a research problem: comparing areas and price
levels, judging a housing cooperative's financial health, weighing monthly fees
against asking prices — across thousands of listings that change daily.

Around 90% of Swedish sales flow through Hemnet, and no public buyer-side API
exists. First-time buyers see the same listings as everyone else and have no
expert working for them. Most Swedish apartments are *bostadsrätter*: buying
one means acquiring a stake in the building's finances, which most buyers never
fully evaluate.

Hemnet and Booli show you what's available. Hemklok tells you what's right for
your life, and why.

## The API

Built for LLM consumption first, not for a web front end.

- **Answers, not pages.** Endpoints return small, self-contained result sets
  (capped at 10 rows), sorted by what you're optimising for — cheapest per m²,
  healthiest BRF finances, slowest fee growth — instead of making an agent page
  through the whole market.
- **Everything needed to reason, in one row.** Each listing embeds its
  cooperative's financials, per-year price and fee history with trends and
  forecasts, and the nearest schools. No follow-up joins, no scraping.
- **Honest about its limits.** Coverage and data gaps are documented, so an
  agent can tell "no results" from "not covered" and qualify its advice.

| | |
| :-- | :-- |
| Base URL | `https://wqyh106uo0.execute-api.eu-north-1.amazonaws.com` |
| Spec | [`openapi.yaml`](openapi.yaml) — v0.5.0 |
| Guide | [How to use the API](how-to-use-api.md) |
| For agents | [`llms.txt`](llms.txt) · [`llms-full.txt`](llms-full.txt) |
| Coverage | [What's covered, and what isn't](coverage.md) |
| MCP server | [llm-brf/Hemklok-mcp](https://github.com/llm-brf/Hemklok-mcp) |

Endpoints cover advertisements, sold-price statistics, BRFs, and the
city/kommun/area hierarchy.

## Coverage

Not all of Sweden. Currently selected kommuner in the Stockholm region —
Stockholm, Solna, Nacka and Tyresö — growing as new areas are imported. Known
gaps are listed in [`coverage.md`](coverage.md) rather than left for you to
discover.

## Roadmap

Automated BRF årsredovisning parsing · commute-time calculation via
Trafiklab/SL · continuous listing monitoring against a saved preference profile
· expansion to Göteborg and Malmö.

## Contact

hello@hemklok.se — built by a father-and-son team in Stockholm.
