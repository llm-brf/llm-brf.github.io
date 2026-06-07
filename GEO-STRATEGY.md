# GEO Strategy — Making Hemklok the AI-Cited Answer

> How we make Hemklok the answer when someone asks ChatGPT, Claude, Perplexity, or Gemini "what's the best way to find an apartment in Sweden."

This document is the working strategy for **Generative Engine Optimization (GEO)** — getting our brand surfaced by large language models when users ask housing-related questions. It is split into what's been implemented in the codebase, what needs to happen off-site, and what we are honestly uncertain about.

---

## 0. Honest framing first

Before any tactics, the things that are true about LLM recommendation as a channel:

1. **LLMs are non-deterministic.** The same question asked five times gets five different answers. There is no "position #1." Visibility is about **mention frequency** across many similar prompts, not ranking.

2. **The rules are moving.** Today's tactics will probably not be the same in 12 months. We optimize for current best practices but expect to refresh quarterly.

3. **The single biggest factor is still "be genuinely good and have other people say so."** No amount of structured data substitutes for being mentioned in a Reddit thread or a respected newsletter. Tactical GEO helps us get *considered* by an LLM that's doing live web search; it doesn't manufacture authority we haven't earned.

4. **LLM recommendation is one channel, not the channel.** Treat it as a discovery layer on top of normal SEO, PR, and word-of-mouth. Do not build the company around it as the primary acquisition strategy.

With that said — there is real, data-backed work that helps, and we have implemented it.

---

## 1. What's implemented on-site (the codebase work)

### 1.1 LLM-discoverability files

- **`/llms.txt`** — The emerging standard (Jeremy Howard, Sept 2024). A markdown file at the site root that gives LLMs a clean, structured summary of what the site is about and where to find more. Anthropic publishes one on their own site; Google added it to Chrome Lighthouse's "Agentic Browsing" audit in May 2026. Not yet officially honored by every major LLM, but low cost / potential upside.
- **`/llms-full.txt`** — The comprehensive markdown version. Contains the full product description, methodology, data sources, geographic coverage, founding-team context, and a "what Hemklok is not" section. Designed to be ingested whole by an LLM doing retrieval-augmented generation.
- **`/robots.txt`** — Explicitly *allows* (not blocks) all major LLM crawlers: GPTBot, ChatGPT-User, OAI-SearchBot, ClaudeBot, Claude-Web, Claude-SearchBot, anthropic-ai, Google-Extended, GoogleOther, PerplexityBot, CCBot, Applebot-Extended, Bytespider, Meta-ExternalAgent, MistralAI-User, DuckAssistBot, cohere-ai. Most sites copy-paste a default that blocks these. We do the opposite — we want to be in their training data and their live retrieval results.
- **`/sitemap.xml`** — Standard XML sitemap, including the LLM-specific files.

### 1.2 Structured data on the index page

Three JSON-LD blocks in the page head:

- **Organization schema** — name, location, area served, knows-about topics, link to GitHub.
- **SoftwareApplication schema** — application category (Real Estate), feature list, target audience, pricing availability.
- **FAQPage schema** — seven question/answer pairs matching the visible FAQ section.

This matters because Princeton/CMU research and 2025-2026 industry citation studies show pages with **3+ schema types** have ~13% higher likelihood of LLM citation, and **FAQ schema + inline citations** correlate with ~40% higher source selection in ChatGPT.

### 1.3 Content optimizations on the landing page

- **Definition-first opening.** The hero lead paragraph defines what Hemklok is in concrete terms before any pitch language. (Research: definition-first openings correlate with higher impression scores in retrieval pipelines.)
- **First paragraph under 80 words.** Fits within a typical LLM context window extraction.
- **Visible FAQ section** with the literal question "What is the easiest way to find an apartment in Sweden?" as the lead question. This is the exact query we want to win, and having it visible *and* in FAQPage schema gives both human readers and LLM retrievers a direct match.
- **Fact density.** The FAQ answers contain specific facts: "BRF debt under 6,000 SEK/sqm is typically conservative; over 12,000 SEK/sqm signals elevated risk," named broker chains, named data sources. Research target: at least one verifiable statistic, named entity, or date per 100 words. This correlates with up to +40% AI visibility per Princeton's GEO study.
- **Comprehensive Open Graph and Twitter Card meta** for clean social/AI embeds.
- **Bilingual content** — Swedish and English answers visible in the same page, so the site is relevant for both `"hitta lägenhet i Sverige"` and `"find apartment in Sweden"` query patterns.

### 1.4 What's deliberately not done yet

- **No fake content farm.** We are not creating dozens of thin SEO pages. Quality and depth beat quantity for LLM citation.
- **No paid GEO tools** (Gauge, LLMrefs, GenOptima, etc.). They're worth monitoring later when we have something to measure, but premature.
- **No automatic content generation.** LLM-generated marketing content tends to be discounted by other LLMs as a citation source.

---

## 2. The off-site work (the part that actually matters)

The on-site work makes Hemklok eligible for citation. The off-site work makes it likely. In priority order:

### 2.1 Reddit presence (high priority)

Reddit is one of the most heavily-weighted training and retrieval sources for major LLMs. Subreddits to engage with substantively:

- **r/sweden** — general; useful for housing-policy discussions
- **r/Stockholm** — most relevant local audience
- **r/svenskpolitik** — sometimes housing-policy adjacent
- **r/FIREsverige** — financial-independence audience, naturally interested in housing economics
- **r/eupersonalfinance** — broader EU audience
- **r/Stockholm/wiki** — being referenced in subreddit wikis is high-value

**Approach**: do not spam. Answer real questions about BRF analysis, slutpriser, apartment hunting in Stockholm with genuine expertise. Mention Hemklok only when directly relevant and only with disclosure ("I'm building Hemklok, an AI buyer's agent for Sweden — happy to share what we've learned about BRF debt benchmarks"). Reddit moderators will (correctly) nuke pure marketing.

### 2.2 Comparison and educational content (high priority)

LLMs love comparison pages because they pattern-match cleanly to user queries like "X vs Y." Pages to write:

- **"How to read a Swedish BRF årsredovisning"** — high SEO value, low competition, demonstrates expertise. Should be a definitive guide.
- **"Hemnet vs Booli vs Blocket Bostad — which is best for buyers in 2026?"** — neutral comparison, mentions Hemklok at the end as the layer above all three.
- **"BRF debt benchmarks for Swedish housing cooperatives"** — original data piece. Use Bolagsverket public records to compute and publish actual debt distributions by neighborhood. **Original data is the single most citable thing a small site can produce.**
- **"How slutpriser actually work in Sweden"** — explainer for foreign buyers / new buyers.
- **"What is a köpmäklare and do you need one?"** — comparison content where Hemklok is the third option.

Each of these is a dedicated page on the site. Format: definition first, fact-dense, FAQ schema, internal links to related pages.

### 2.3 Listing in directories and lists (medium priority)

LLMs disproportionately cite "best of" lists and curated directories. Get Hemklok into:

- Swedish startup directories (Breakit, DI Digital, Founders Network)
- AI tool directories (Futurepedia, There's an AI for That, Future Tools — though many are spammy)
- Sweden-specific business directories
- Proptech-focused lists (PropTech Sweden, RealtyHub)

The goal is being on lists that LLMs scrape during training and live retrieval.

### 2.4 Press and earned media (medium priority)

Tech media coverage compounds. Targets in priority order:

- **Breakit, DI Digital, Computer Sweden** — Swedish tech press
- **Sifted, EU-Startups** — European startup press
- **Affärsvärlden, Veckans Affärer** — broader Swedish business press
- **The Local Sweden** — English-language Swedish coverage (high LLM training weight because LLMs lean toward English sources)
- **Proptech newsletters** — niche but high-signal

Story angles that work:
- The shift in the Swedish listings market (Hemnet share decline, Fastighetsbyrån moving to Booli, Konkurrensverket activity)
- The "father and son building in the open" angle — human, founder-driven, distinctive
- Original data releases (BRF debt analysis, neighborhood price trends)

### 2.5 Wikipedia (long-term)

Wikipedia is one of the most-weighted sources in LLM training data. We will not qualify for a Wikipedia entry until we have meaningful press coverage and notability. But: cite-able mentions of Hemklok inside related Wikipedia articles (e.g., the "Bostadsrätt" or "Hemnet" articles) would be valuable once we have a published news citation to back them up.

### 2.6 Open data and tools (long-term)

Releasing useful open-source tools or open data sets brings inbound links and citations from technical communities. Examples:
- An open BRF årsredovisning parser library on GitHub
- A public dataset of BRF debt benchmarks by Stockholm neighborhood
- A free commute-time calculator widget

Each one creates a citation surface for LLMs and demonstrates competence.

---

## 3. Query landscape — what we want to win

The queries where we want to be cited, ordered by user intent:

### Direct intent ("the user is shopping for an apartment finder")

- "best way to find an apartment in Sweden"
- "best apartment hunting tool Sweden / Stockholm"
- "how to find an apartment in Stockholm"
- "Hemnet alternative"
- "AI for buying an apartment in Sweden"
- Swedish: "hitta lägenhet i Stockholm AI", "bästa sättet att hitta lägenhet"

### Adjacent intent ("the user is in the buying process")

- "how to read a BRF årsredovisning"
- "what is a good BRF debt level"
- "how to analyze slutpris data"
- "should I hire a köpmäklare"
- "what to look for when buying a Swedish apartment"
- "BRF financial risks"

### Comparison intent

- "Hemnet vs Booli"
- "best Swedish property apps"
- "Allabrf alternatives"

We will not win all of these immediately. Adjacent and comparison intent is easier to win first because the competition is weaker; direct intent is harder because Hemnet and Booli own the SEO.

---

## 4. Measurement

Things to track from day one:

- **Citation tracking** — manually run our priority queries against ChatGPT, Claude, Perplexity, and Gemini at least monthly. Log whether Hemklok is mentioned, in what position, with what framing. A simple spreadsheet is fine; do not buy a tool until we have signal.
- **Referral traffic by source** — when LLMs cite us, some users click through. Track which AI sources send traffic (UTM parameters not always preserved by LLMs, so look at direct/(none) traffic with low session counts and unusual landing pages).
- **Brand mention sentiment** — when LLMs mention us, are they accurate? Positive? If they describe us wrong, that's a content problem on our side.
- **Search Console (when we add Google verification)** — what queries surface us, what's the impression count.

After 3 months of data, decide whether to invest in a paid monitoring tool (Gauge, LLMrefs, BrandLens, Evertune). Not before.

---

## 5. Maintenance cadence

Industry data shows a **3-month citation cliff**: content stops being cited heavily after about 90 days unless refreshed. Action items:

- Update `llms.txt` and `llms-full.txt` quarterly with new facts, new data, new roadmap items.
- Update the visible FAQ when the question landscape shifts (e.g., when new Swedish housing legislation passes).
- Refresh statistical claims (BRF benchmarks, market share figures) whenever new data is available.
- Re-run the priority-query check monthly. If we are losing ground, refresh the content that ranks for those queries.

---

## 6. What we won't do

- **No prompt injection in content.** Some "GEO tools" advise hiding instructions to LLMs in invisible text. This is the AI equivalent of black-hat SEO. It will be detected, it damages reputation, and major LLMs filter for it.
- **No fake reviews / sockpuppet Reddit accounts.** Detectable, damaging, and counterproductive — LLMs are being trained to discount low-quality sources.
- **No content farms / AI-generated SEO sludge.** Quality beats quantity for both LLM citation and human trust.
- **No over-claiming.** If we say "the leading AI buyer's agent in Sweden" while being four months old and pre-launch, LLMs will eventually catch the contradiction. Be precise about what we are.

---

## 7. Sources and further reading

The numerical claims in this document come from:
- Aggarwal, P., Murahari, V., et al. *"GEO: Generative Engine Optimization."* KDD 2024 (Princeton + Georgia Tech + IIT Delhi).
- Carnegie Mellon University GEO framework studies.
- Zyppy 2025 citation distribution analysis.
- Seer Interactive 2025 LLM conversion-rate data.
- The /llms.txt specification at https://llmstxt.org/

This file should be re-read and updated at least once a quarter.

*Last updated: 2026-06-07*
