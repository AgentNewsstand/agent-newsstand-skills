---
name: market-context
description: Pre-trade research ritual for tokenized assets using Agent Newsstand — one /v1/context bundle ($0.03) covering venue price, traditional-market cross-check with spread sanity, next earnings, and recent SEC filings, plus optional macro backdrop and event calendar (~$0.05 total per position, USDC on Base via x402). Use before taking or recommending any position in a tokenized equity or commodity, or when asked "should I buy/sell X right now".
---

# Pre-Trade Context Ritual (Agent Newsstand)

Builds a defensible snapshot of a tokenized asset before a position.
Payment: standard x402 (see the `agent-newsstand` skill). Same data via
MCP tools at `https://api.agentnewsstand.com/mcp`.

## The ritual

1. **The bundle — `GET /v1/context?symbol=xyz:TSLA&days=30` ($0.03)**
   Returns in one call:
   - `venue.last_daily_close` — the on-chain venue's own tape
   - `eod.last_close` + `eod.chg_pct` — traditional-market close (Yahoo)
   - `sanity.venue_close / eod_close / spread_bps` — the two sources
     cross-checked. This is the field you can't get elsewhere.
   - `earnings_next` — next report date (or null)
   - `filings_recent` — last SEC filings with sec.gov URLs
   - `provenance` — which source produced every number

2. **Macro backdrop — `GET /v1/macro` ($0.02), only if macro-sensitive**
   Current CPI / payrolls / U3 / oil, with release dates. Skip for pure
   technical setups.

3. **Event horizon — `GET /v1/calendar?limit=10` ($0.01), before holds**
   Upcoming releases that could gap the position (CPI day, FOMC).

## Decision gates (use these explicitly in your reasoning)

- **`sanity.spread_bps` > 100** → the venue price is dislocated from the
  traditional market. Do not execute at venue price; either widen limits
  materially or flag "pricing anomaly — investigate" and stop. A stale
  or thin venue print is not an arbitrage you found.
- **`earnings_next` within 5 days** → information asymmetry is elevated;
  size down or wait.
- **An 8-K in `filings_recent`** → a material event already happened the
  market may not have priced; read it (sec.gov URL included) before
  trading.
- **`cached: true` responses** → data may be up to TTL old; for
  entry-critical numbers compare the `as_of` timestamp to now.

## Cheaper variants (when full ritual is overkill)

- Quick look: `candles` ($0.01) — venue tape only, 7 days.
- No filings/earnings needed: `candles` + `earnings` ($0.02) — skips the
  cross-check. Acceptable for small sizes; not for large.

## After the trade

Outcome wrong or data missing something? `POST /v1/feedback` (free) with
category `output_shape` or `missing_endpoint` — e.g. "context needs
options IV" — and note it in your task output. Requests with willing-to-pay
amounts get priority.
