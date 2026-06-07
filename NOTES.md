# Hemklok — Project Notes

> An AI buyer's agent for the Swedish apartment market. This document is the living scratchpad: vision, philosophy, data sources, architecture, roadmap, open questions.

---

## 0. Naming

Working name: **Hemklok** (Swedish: *hem* = home, *klok* = wise/smart).

Alternatives considered (kept here so we can swap with a find-and-replace if we change our minds):

- **Hemklok** — chosen. Pure Swedish, distinctive, low SEO collision, reads cleanly in English too.
- **Klok** — minimalist, but too generic as a standalone word.
- **Bovis** — "bo" (live/dwell) + wisdom suffix. Decent but invented-sounding.
- **Hembas** — too close to existing brands.
- **Smart Choices** — original suggestion. Generic, very crowded SEO-wise, weak trademark position.
- **Nestor** — wise advisor in Greek myth. International but loses the Swedish grounding.

Domain: aim for `hemklok.se` as primary, `hemklok.com` as a defensive secondary.

---

## 1. The Core Vision

We are **not** building another listings website. Hemnet and Booli already exist; we will lose if we try to outdo them at their own game.

We are building a **personal buyer's agent** — the AI equivalent of hiring someone to do the work of finding, vetting, and evaluating apartments on your behalf. The user describes their life; we do the boring, careful, expert work; they make a better decision with less stress.

The bet:
- **Search is commoditized.** Hemnet already lets you filter by rooms and price.
- **Judgment is not.** Reading a BRF årsredovisning, weighing commute against rent, spotting that a renovation assessment is coming — that's where humans currently fail and where a careful AI can win.
- **Continuity matters.** A search is a moment; a buyer's agent is a relationship over weeks or months. The product should *remember* you and improve as it learns your real preferences.

---

## 2. The Core Agent Philosophy — Preferences → Inferences → Constraints

This is the heart of what makes Hemklok different from a filter form.

A traditional listings site asks: *how many rooms? what price?*
A real buyer's agent asks: *what does your life look like?* — and then translates that into the constraints you wouldn't have thought to specify yourself.

### The inference layer

When the user says **"I have a dog"**, the agent should not just check a "pets allowed" box. It should infer:

- **Outdoor needs**: dog park within walking distance OR direct access to forest/park (high priority for a Stockholm dog)
- **Apartment size**: probably needs at least a small extra surface beyond what a single human needs
- **Floor & elevator**: ground floor or elevator strongly preferred — daily walks make stairs with a dog (especially old/large) miserable
- **BRF rules**: not all BRFs permit dogs; check the stadgar
- **Flooring**: hardwood is harder on dogs and noisier for neighbors; should flag
- **Neighbor proximity**: end units / corner apartments score higher for noise tolerance
- **Distance to vet**: a low-priority but real signal if the user mentions an older or anxious dog

The user said three words. The agent surfaced ten constraints. **That's the product.**

### More examples of the inference pattern

| User says | Agent infers |
|---|---|
| "I work from home" | Quiet street > main road; dedicated workspace = bigger floor plan; good light = direction & windows; reliable fiber = check BRF infrastructure |
| "I have small kids" | School catchment area; preschool (förskola) availability; playground proximity; non-toxic renovation history; balcony safety; storage room for strollers |
| "I'm single, social life is downtown" | Late-night transport access matters more than commute; smaller apartment OK; cultural amenity density |
| "I bike everywhere" | Bike storage (cykelförråd) in BRF; bike-friendly streets; elevation profile to common destinations |
| "I might have kids in a few years" | This is a 5-7 year purchase, not a 2-year one. Different threshold for school zones and apartment growth-room. |
| "Investment property, won't live there" | Optimize for resale liquidity; popular neighborhoods over personal-fit; rental rules in BRF; tax/avgift structure |
| "I'm allergic" | No cat in current owner's possession; renovation history; ventilation system age; carpet vs hardwood |

### Implementation sketch

The inference layer is a structured reasoning step between the conversational turn and the listing filter. Roughly:

```
User utterance
    ↓
Extract explicit signals (rooms, price, area)
    ↓
Extract implicit life-context signals (dog, kids, WFH, etc.)
    ↓
Map signals → weighted constraints (some hard, some soft, some scoring-only)
    ↓
Confirm inferences with the user ("It sounds like you'd also want X — does that fit?")
    ↓
Persist as user model — refine over time
```

The "confirm inferences" step is critical. The agent should not silently impose constraints; it should surface them and let the user accept, reject, or modify. This builds trust and makes the user model more accurate over time.

---

## 3. Data Sources

### Tier 1 — Listings (the hardest problem)

**Hemnet** — ~90% of Swedish home sales, but no public/partner API. They are protective and historically litigious. Their listing share is currently dropping (Q4 2025 listings -26.4% YoY; Fastighetsbyrån moved to Booli in mid-2026). EU Database Directive likely grants them `sui generis` rights over their compilation; do **not** plan a business around scraping them as the primary source.

- **Strategy**: user-mediated access. The user has the Hemnet account; we provide a tool that helps them organize and analyze listings *they* have access to (saved searches, forwarded listings, browser extension). Legally this is a different posture than us scraping the site.

**Booli** — owned by SBAB (state). No public API but historically more partnership-open. Now has Fastighetsbyrån's listings. Worth a direct B2B conversation before assuming we have to work around them.

**Direct broker chains** — our most realistic primary path. Many publish XML/JSON-LD feeds on their own sites:
- Fastighetsbyrån
- Svensk Fastighetsförmedling
- Notar
- HusmanHagberg
- SkandiaMäklarna
- Mäklarhuset
- Erik Olsson
- Bjurfors
- Alvhem (high-end Stockholm)
- Lagerlings (high-end Stockholm)

Their incentives align with ours: they want listings distributed, they want qualified leads. The pitch is "qualified buyers in exchange for early/clean data access."

### Tier 2 — Public records (where we win on depth)

- **Bolagsverket** — BRF annual reports (`årsredovisning`) are legally required and public. This is the moat for our analysis feature. Allabrf and Hittabrf have done this — source is open, we can build our own.
- **Lantmäteriet** — property identifiers (`fastighetsbeteckning`), boundaries, ownership history. Geodatakatalogen has open data; some paid APIs.
- **Skatteverket / lagfart** — historical sale prices (slutpris). Public records, harder to access cleanly than Hemnet's display.
- **SCB (Statistics Sweden)** — neighborhood demographics, income, household composition.
- **Stockholm Stad open data portal** — schools, daycare, crime, **planned construction (`detaljplaner`)** — the latter is huge for future-value modeling.
- **Other municipalities** — most have similar portals (Solna, Nacka, Lidingö, Sundbyberg, etc. for greater Stockholm).

### Tier 3 — Context APIs

- **Trafiklab / Samtrafiken** — free, official API for ALL Swedish public transport (SL, SJ, regional). This is our commute feature. Better than Google Maps for transit because it's the actual source.
- **Riksbanken** — interest rate data, policy rates.
- **SCB housing price indices** — official market trend data.
- **Bank rate APIs** — SBAB, Swedbank, Handelsbanken, Nordea, SEB, Länsförsäkringar publish list rates + actual snittränta. Compricer aggregates.
- **Weather / climate** — for solar exposure modeling on apartment direction.

### What we will NOT do (yet)

- No Hemnet scraping as primary infrastructure (legal exposure too high)
- No reselling raw listing data (we are not a data broker; we are an advisor)
- No storing broker personal data beyond what's necessary for the user's specific outreach (GDPR)

---

## 4. Architecture Sketch (high level)

```
┌─────────────────────────────────────────────────┐
│  Conversational interface (web / iOS / Android) │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│  Agent orchestrator                             │
│  - conversation memory                          │
│  - user preference model (persistent)           │
│  - inference layer (signal → constraint)        │
│  - tool routing                                 │
└────────────────────┬────────────────────────────┘
                     │
        ┌────────────┼─────────────┬──────────────┐
        ▼            ▼             ▼              ▼
   ┌─────────┐  ┌─────────┐   ┌─────────┐   ┌─────────┐
   │Listings │  │ BRF     │   │Commute  │   │Market   │
   │ sources │  │analysis │   │engine   │   │intel    │
   └─────────┘  └─────────┘   └─────────┘   └─────────┘
        │            │             │              │
        ▼            ▼             ▼              ▼
   Broker feeds  Bolagsverket  Trafiklab     SCB, banks,
   Booli         (årsred.)     SL            Riksbanken
   Aggregators   (PDF parse)
```

Key design notes:

- **Agent loop, not request/response.** The orchestrator runs continuously per user — new listings trigger evaluation against the persistent preference model, not just live queries.
- **Tool-using LLM as the brain.** Each capability (BRF analyzer, commute engine, etc.) is a tool the agent can call. This is how we get from "chat with a search engine" to "agentic buyer's helper."
- **Preference model is durable.** Stored separately from conversation history. Survives sessions. Improves with feedback.
- **Human-in-the-loop for outbound actions.** Anything that contacts a broker or schedules a viewing must be approved by the user before being sent. Trust is the product.

---

## 5. Feature Roadmap

### Phase 1 — Foundation (now)
- Conversational preference capture + inference layer
- BRF årsredovisning ingestion + analysis (PDF parsing, financial health scoring)
- Manual listing import (paste a Hemnet URL, we analyze)
- Basic ranking & comparison

### Phase 2 — Coverage (next ~3 months)
- Broker feed integrations (start with 2–3 chains)
- Booli partnership conversation
- Commute engine via Trafiklab
- Historical price analysis (slutpriser + trend modeling)

### Phase 3 — Autonomy (next ~6 months)
- Continuous monitoring + smart notifications (only fire when something genuinely beats the existing shortlist)
- Broker outreach drafts (user approves before sending)
- Viewing logistics (calendar integration, reminders)
- Generated viewing checklists, personalized per apartment

### Phase 4 — Decisions (next ~9–12 months)
- Negotiation guidance grounded in comparable sales + BRF posture
- Future-value modeling using `detaljplaner` + neighborhood trends
- Multi-property side-by-side comparisons with trade-off analysis

### Phase 5 — Seller side (2027)
- Honest pricing recommendations
- Listing optimization guidance
- The "AI staging advisor" pattern
- Long-term: the marketplace itself

---

## 6. Legal & Compliance Notes

- **GDPR**: any personal data (broker contacts, user emails, BRF member info incidentally in årsredovisningar) needs lawful basis + retention policy from day one. Cheaper to design in than retrofit.
- **EU Database Directive**: protects substantial database compilations even if individual data points are public. This is the legal theory under which Hemnet would come after scrapers. We respect it.
- **Konsumentköplagen / fastighetsmäklarlagen**: if we ever cross into recommending purchases or contacting brokers on behalf of users, we need to understand where the line is between "assistant" and "regulated broker activity." Talk to a Swedish lawyer before scaling outbound actions.
- **AI Act (EU)**: high-risk classification probably doesn't apply to us, but transparency obligations do (users should know they're talking to AI; outputs should be inspectable). Build with this in mind.
- **Robots.txt and ToS**: where we do automated access, we respect robots.txt and rate limits. Public-data scraping in EU law is messier than in US — the "if it's in incognito, it's fair" rule does **not** map cleanly here.

---

## 7. Open Questions (things we don't have answers to yet)

1. **What's the willingness to pay?** Subscription? One-time fee per successful purchase? Free with broker commission share? This affects everything.
2. **How do we measure success?** "Helped you decide" is fuzzy. NPS? Time-to-purchase? User-reported regret six months in?
3. **Cold start problem for new users.** The preference model gets better with use, but the first conversation has to feel useful too. How do we make the demo magical?
4. **Multi-user households.** Most apartment purchases are couples or families. Whose preferences? How do we mediate disagreement? (This is actually a feature opportunity — a couples-mediation tool.)
5. **Renting vs buying.** Sweden has unique rental market dynamics (förstahandskontrakt, kötid, andrahand). Do we start with sales only, or include rental from day one?
6. **Geography expansion.** Stockholm first, obviously. Then Gothenburg/Malmö? Or smaller Stockholm-adjacent (Solna, Nacka, Lidingö) first? The latter shares the broker ecosystem.
7. **Brokerage relationship.** Are we their friend (helping them sell) or their adversary (helping buyers see through them)? The answer is probably "both, situationally" but the positioning matters.

---

## 8. Working principles

- **Honesty over optimism.** If an apartment is overpriced, we say so. If a BRF is in trouble, we say so. Users will trust us *because* we don't always tell them what they want to hear.
- **Slow over fast.** Buying a home isn't a panic decision. The product should encourage thinking, not impulse.
- **Specific over generic.** "This BRF's debt-per-square-meter is in the 90th percentile for the neighborhood and they have a planned facade renovation in 2027" beats "good investment potential" every time.
- **Show your work.** Every recommendation links back to the data that produced it. No black-box scoring.
- **Build in the open.** Father and son, public roadmap, honest about what's working and what isn't.

---

*Last updated: working draft. This document is a snapshot — update freely as we learn.*
