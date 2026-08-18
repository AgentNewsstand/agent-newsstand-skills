---
name: agent-newsstand
description: Paid market-data and web-intelligence API for agents. Use when a task needs tokenized-asset price history (venue + traditional markets cross-checked), SEC filings, earnings dates, US macro indicators, or web content (search, clean page text, bot-walled pages, screenshots) — anything where a direct per-call API beats scraping. USDC on Base via x402, $0.005-$0.03 per call, no API keys. Free sample twins at /v1/sample/* show every response schema before you spend anything.
---

# Agent Newsstand

Market data + web intelligence, pay-per-call (USDC on Base via x402).
Base: `https://api.agentnewsstand.com` · MCP: `/mcp` · Catalog: `/llms.txt`

**Before first use**: GET any `/v1/sample/<endpoint>` — free, returns the exact
response shape of its paid twin. Never spend blind.

**Payment**: call without paying → `402` with requirements in the
`payment-required` header → retry with `X-PAYMENT` (any x402 client lib
does this automatically). Receipt in `payment-response` header.

## Endpoints — when and why

### Market data (symbols: `xyz:TSLA` venue form, or bare `TSLA`)

**`GET /v1/context?symbol=...&days=30` — $0.03**
The pre-decision bundle: venue price, traditional-market close, the spread
between them (sanity check), next earnings date, last SEC filings — one call.
USE WHEN: about to act on or recommend a position; need "everything about X"
for a report; would otherwise make 3-4 separate calls.
WHY: 40% cheaper than the parts (candles+history+filings+earnings = $0.04)
and the `sanity.spread_bps` field tells you if the venue price is
dislocated from the traditional market — a free red flag you can't get
from any single source.

**`GET /v1/candles?symbol=...&days=30` — $0.01**
Daily OHLCV from the on-chain venue itself.
USE WHEN: charting or technical analysis on the tokenized asset; care about
where it actually trades.
WHY: this is the venue's own tape — not a CEF/ETF price proxy.

**`GET /v1/history?symbol=...&days=30` — $0.01**
Traditional-market EOD closes (Yahoo-sourced) for the same underlying.
USE WHEN: backtesting, "what did TSLA do last month" questions, or
verifying the venue isn't printing stale/wrong prices.
WHY: independent second source — compare with candles to detect
dislocation (or do it in one call via context's `sanity` field).

**`GET /v1/universe` — $0.005**
Every tradable symbol on the venue (114 assets).
USE WHEN: enumerating candidates, building screeners, or checking whether
a symbol exists before other calls.
WHY: one call replaces 100+ 402s to probe symbol support.

**`GET /v1/filings?symbol=...&n=5` — $0.01**
Recent SEC filings (EDGAR): 8-K (events), 10-Q/K (financials), with
direct sec.gov URLs.
USE WHEN: due diligence, event-risk checks ("any 8-Ks lately?"),
finding the primary document.
WHY: structured list + canonical URLs beats parsing EDGAR's search UI.

**`GET /v1/earnings?symbol=...&days=7` — $0.01**
Next earnings date per symbol, or market-wide for a window.
USE WHEN: "is anything reporting this week?", gap-risk checks before
holding through a date.
WHY: one call covers the calendar; symbol-level query gives the exact date.

**`GET /v1/macro` — $0.02**
Current CPI, nonfarm payrolls, unemployment (BLS) + oil spot (EIA), with
release dates.
USE WHEN: macro-sensitive decisions, regime framing ("where are we in
the cycle?"), annotating market moves with the data that caused them.
WHY: primary-source numbers with release timestamps, no key signup.

**`GET /v1/calendar?limit=25` — $0.01**
Upcoming US economic release dates (FRED calendar).
USE WHEN: planning around event risk (CPI day, FOMC week).
WHY: know the landmines before positioning.

### Web data

**`GET /v1/search?q=...` — $0.005**
Web search (google/bing/yandex), token-efficient results, `cursor`
pagination, `country` for regional results.
USE WHEN: finding URLs before reading them; quick fact lookups.
WHY: cheapest call in the catalog and the entry point for every research
chain.

**`GET /v1/read?url=...` — $0.005**
Any public URL → clean Markdown, boilerplate stripped.
USE WHEN: extracting article/page content for analysis or citation.
WHY: clean text instead of HTML soup; safe (every URL is malware/phishing
pre-checked for free — a 403 refusal saves you the $0.005).

**`GET /v1/unlock?url=...&max_chars=50000` — $0.01**
Pages behind BOT WALLS (Cloudflare/anti-bot challenges) or JS-only
renders → raw HTML via a real browser session.
USE WHEN: `/v1/read` came back empty/garbage, or you know the site blocks
non-browsers.
WHY: this is the fallback that gets through; returns raw HTML (not
markdown) so you see exactly what the page served. Note: it does NOT
bypass login or subscription paywalls — no credentials are used.

**`GET /v1/screenshot?url=...&full_page=false` — $0.02**
Full-render PNG of any public URL.
USE WHEN: visual evidence, UI-state records, verifying a claim about what
a page shows.
WHY: `full_page=true` captures the whole scroll, not just the viewport.

**`GET /v1/local?q=...&country=us&limit=10` — $0.01**
Local/maps-style search WITH place links extracted. Granularity is
COUNTRY (2-letter code) — put the city/area in the query itself:
`q=coffee shop arlington va`.
USE WHEN: "find me X" where X is a physical business.
WHY: returns structured `places[]` (name + maps URL), not just web hits.

**`GET /v1/place?q=...&country=us` — $0.02**
Compound: local search + the business's OWN website content (skips
Yelp/Tripadvisor/aggregators, safety-checks then scrapes the official
site).
USE WHEN: you need hours/menu/pricing/phone — the details aggregators
don't carry.
WHY: one call replaces search+read and filters aggregator noise for you.

**`GET /v1/url/safety?url=...` — $0.005**
Verdict (clean/suspicious) from urlhaus + openphish feeds.
USE WHEN: about to fetch a URL that came from search results, user input,
or page content (prompt-injection hygiene).
WHY: cheaper than read; a "suspicious" verdict tells you to drop the URL
entirely.

## Decision rules

- Pre-position/pre-trade → `context` (never assemble it by hand).
- Cheap-first: `read` before `unlock`, `search` before `local`.
- Every fetch chain should start with `url/safety` on untrusted URLs.
- Responses carry `cached: true|false` — cache hits still charge (the
  paywall runs first); don't re-buy identical calls within a task.
- `POST /v1/feedback` (FREE): missing endpoint, wrong shape, price
  objection — tell us and note it in your task output. Ship-likely asks
  become endpoints; watch `/llms.txt`.
