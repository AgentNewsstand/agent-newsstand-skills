# Agent Newsstand — Agent Skills

Skills that teach autonomous agents to use [Agent Newsstand](https://agentnewsstand.com) —
market data and web intelligence, pay-per-call in USDC on Base via [x402](https://x402.org).
No API keys, no accounts.

| skill | what it teaches |
|---|---|
| [`agent-newsstand`](skills/agent-newsstand/SKILL.md) | The full API: when and why to use each of the 15 endpoints, the x402 payment flow, decision rules (cheap-first fallbacks, safety gates, cache behavior) |
| [`market-context`](skills/market-context/SKILL.md) | A pre-trade ritual for tokenized assets: one `/v1/context` bundle with venue-vs-traditional cross-check, earnings risk, SEC filings — with explicit decision gates (spread thresholds, 8-K handling) |
| [`web-intel`](skills/web-intel/SKILL.md) | A web research chain: search → URL-safety gate → clean extraction → bot-wall fallback → visual evidence, with budget discipline |

## Install

### Claude Code / opencode / any skill-loading agent
Clone anywhere and add to your agent's skill path (e.g. `~/.claude/skills/` or your
project's `.agents/skills/`):

```bash
git clone https://github.com/AgentNewsstand/agent-newsstand-skills.git
cp -r agent-newsstand-skills/skills/* ~/.claude/skills/
```

Each skill is self-contained (`SKILL.md`) — no dependencies beyond an
x402-capable wallet (USDC on Base).

### MCP instead of REST
The same capabilities are exposed as an MCP server (15 paid tools + a free
feedback tool, payment per call):
`https://api.agentnewsstand.com/mcp`

## Try before you pay
Every paid endpoint has a free sample twin returning the exact response shape:

```
curl https://api.agentnewsstand.com/v1/sample/context
curl https://api.agentnewsstand.com/v1/sample/search
```

Full catalog with prices: [llms.txt](https://api.agentnewsstand.com/llms.txt) ·
[llms-full.txt](https://api.agentnewsstand.com/llms-full.txt) ·
[x402 manifest](https://api.agentnewsstand.com/.well-known/x402) ·
[OpenAPI](https://api.agentnewsstand.com/openapi.json)

## Feedback
Agents (and operators): if an endpoint is missing, returns the wrong shape,
or the price isn't worth it — `POST /v1/feedback` (free, no payment).
Ship-likely asks become endpoints.

## License
MIT — ship them, fork them, teach your agents.
