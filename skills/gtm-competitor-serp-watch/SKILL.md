---
name: gtm-competitor-serp-watch
description: Track who wins Google and Google AI Mode for your buying keywords, you or your competitors, and what changed since the last check. Use when the user says "who ranks for our keywords", "competitor SERP watch", "share of voice on Google for these keywords", "are competitors in the AI answer", "weekly rank check against competitors". Runs one Google results page and one AI Mode answer per keyword through looot, about $0.003 per keyword.
---

# Competitor SERP watch

For each buying keyword ("best crm for startups", "clay alternative"), reads the Google top 10 and Google's AI Mode answer, records which of the user's and competitors' domains appear and where, and compares with the previous check saved in a JSON file.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- 5 to 50 keywords people type when they are ready to buy.
- The user's domain and up to 10 competitor domains.
- Country and language (default `us`, `en`).
- Where to keep history (default `serp-watch.json` in the working folder).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `google.serp.organic` | `query`, plus `gl` and `hl` | from $0.0009 per call |
| 2 | `google.serp.ai-mode` | `query` | from $0.002 per call |
| 3 (optional) | `search.chatgpt` | `query` | from $0.004 per call |

Expected total: about $0.003 per keyword, $0.15 for 50 keywords, with the optional ChatGPT check about $0.35.

## Quote before you run

1. Keywords × the steps chosen × the prices above.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.02).

## Run it

```tool
run {"endpointId": "job:google.serp.organic", "input": {"query": "clay alternative", "gl": "us", "hl": "en"}, "idempotencyKey": "serp-2026-10-08-clay-alternative", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

```tool
run {"endpointId": "job:google.serp.ai-mode", "input": {"query": "clay alternative"}, "idempotencyKey": "aimode-2026-10-08-clay-alternative", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

Put the date in the idempotency key, so the same day replays for free and the next week fetches fresh data.

For each keyword record: the position (1 to 10) of every tracked domain in the organic results (match on the host, ignore `www.`), and for AI Mode whether each domain is cited as a source or named in the answer text. Then load the previous check from the history file and compute the change.

## Output format

A markdown report, then the updated history file.

```markdown
# SERP watch, 2026-10-08 (us/en), 20 keywords, cost $0.06

## Share of voice
| Domain | Top 3 | Top 10 | Cited in AI Mode | Change vs 2026-10-01 |
|---|---|---|---|---|
| you.com | 4 | 11 | 3 | +2 top 10 |
| rival.com | 9 | 15 | 8 | -1 top 3 |

## Changes
- "clay alternative": rival.com 3 -> 1, you.com 7 -> 12 (dropped out)

## Per keyword
| Keyword | you.com | rival.com | other.com | AI Mode cites |
|---|---|---|---|---|
```

History file shape: `{"checks": [{"date": "2026-10-08", "gl": "us", "keywords": {"<kw>": {"organic": {"<domain>": <pos>}, "aiMode": ["<domain>"]}}}]}`.

## Cost and stop rules

- One results page per keyword; do not page to 20 unless the user asks.
- Two failures on a keyword: mark it `error` in the report.

## When not to use this skill

- One results page: `looot-google-serp`. AI visibility for a brand across engines: `looot-ai-search-visibility`. Both in `loootai/looot-skills`.
