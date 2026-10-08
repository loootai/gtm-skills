---
name: gtm-funding-signal-scan
description: Find companies that just raised money and match your ICP, or check which accounts on a list raised recently. Use when the user says "which startups raised a Series A this month in fintech", "recently funded companies to prospect", "funding signals for my account list", "who just raised in Europe" or "did any of these accounts raise". Searches the news and reads funding rounds through looot, about $0.01 to $0.02 per company.
---

# Funding signal scan

A company that raised in the last 60 days is hiring and buying. This skill works in two modes: discover (search recent funding news for a segment, then confirm each round) or check (take the user's account list and read each company's latest round).

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- Mode: discover a segment, or check a list of domains.
- Discover: the segment in words (`fintech`, `B2B SaaS in Germany`), round types (`seed`, `Series A`), window (default 60 days).
- Check: the domains.

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 (discover) | `news.search` | `query` such as `"raises Series A" fintech` | from $0.00099 per call |
| 2 | `company.enrich` | `domain` or `company` name from the article | from $0.0019 per call |
| 3 | `company.funding` | `domain` | from $0.01 per call |
| 1 (check mode) | `company.funding` | each `domain` | from $0.01 per call |

`signals.funding` (from $0.108 per call) and `company.funded.search` (from $0.05 per result) return ready-made funding feeds. They cost 10 to 100 times more per company; offer them only when the news route finds too little.

Expected total: discover with 5 queries and 30 companies ≈ $0.40. Check mode, 100 domains ≈ $1.

## Quote before you run

1. Discover: queries × $0.001, plus companies found × (step 2 + step 3). Check: domains × $0.01.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.03).

## Run it

```tool
run {"endpointId": "job:news.search", "input": {"query": "\"raises\" \"Series A\" fintech"}, "idempotencyKey": "fund-2026-10-08-fintech-a", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

From each article take the company name, amount, round, lead investor and date. Skip articles older than the window and roundups that name ten companies without amounts. Resolve the domain with `company.enrich`, then confirm with `company.funding`:

```tool
run {"endpointId": "job:company.funding", "input": {"domain": "acme.com"}, "idempotencyKey": "fund-acme.com", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.03}}
```

A round that appears in the news but not in `company.funding` stays on the list with `confirmed: news only`.

## Output format

```csv
company,domain,round,amount_usd,announced,lead_investor,employees,country,source_url,confirmed
Acme,acme.com,Series A,12000000,2026-09-22,Example Ventures,45,DE,https://...,funding+news
```

Sorted by announcement date, newest first. Then counts and the total `actualCost`.

## Cost and stop rules

- Dedupe companies across queries before step 2.
- Two failed queries in a row: stop and report.

## When not to use this skill

- Full company profile: `gtm-account-research-brief`.
