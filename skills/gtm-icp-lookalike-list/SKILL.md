---
name: gtm-icp-lookalike-list
description: Build a target account list from your best customers. Use when the user says "find companies like our top 5 customers", "lookalike accounts for stripe.com and notion.so", "build an ICP list from these seed accounts", "more companies like this one" or "expand my account list". Enriches the seeds, finds similar companies, enriches and scores each candidate through looot, about $0.01 to $0.02 per candidate.
---

# ICP list from lookalikes

Starts from 1 to 5 seed companies the user already wins with, finds similar companies, enriches every candidate and scores it against a profile built from the seeds. The output is a ranked account list with the reason each account is on it.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- 1 to 5 seed domains (best customers, not logos they admire).
- How many accounts they want (default 50).
- Hard filters: countries, employee range, industries to exclude, existing customers to drop (a CSV or a list of domains).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. The provider looot routes to can cost more; `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `company.enrich` | each seed `domain` | from $0.0019 per call |
| 2 | `company.similar` | each seed `domain` | from $0.0089 per call or $0.0019 per result |
| 3 | `company.search` (optional) | filters built from step 1 (industry, size, country) | from $0.0089 per call or $0.00036 per result |
| 4 | `company.enrich` | each candidate `domain` | from $0.0019 per call |

Expected total for 50 accounts: about $0.05 for the seeds, $0.10 to $0.40 for similar-company searches, $0.10 for enrichment. Under $1 with the cheapest providers.

## Quote before you run

1. Work out the number of runs: seeds × 2, plus one enrichment per candidate. Multiply by the prices above or by `costPerSuccessUsd` from `search_catalog`.
2. Tell the user the total and the `balance`, then wait for a yes. More than 100 runs: wait for an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.05 is enough for every step here).

## Run it

```tool
run {"endpointId": "job:company.similar", "input": {"domain": "stripe.com"}, "idempotencyKey": "icp-<list>-similar-stripe.com", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.05}}
```

Inspect `company.similar` and `company.search` once before the first run: some providers take a domain, others a name or filter fields, and live input names win over this file.

Then:

1. Build the profile from the seeds: shared industry, employee range, country, business model (B2B or B2C), and any tech that all seeds share.
2. Merge candidates from all seeds, drop duplicates by domain, drop the seeds, existing customers and anything failing a hard filter. Do this before step 4 so you do not pay to enrich accounts you will drop.
3. Enrich what is left, then score: +2 same industry, +2 employee range within the seeds' range, +1 same country, +1 named by two or more seeds' similar lists. Keep the top N.

## Output format

A CSV the user can import, plus a short summary:

```csv
rank,domain,name,industry,employees,country,score,why,similar_to
1,example.com,Example Inc,Fintech,180,US,6,"same industry, size in range, similar to 2 seeds","stripe.com;adyen.com"
```

Summary lines: seeds used, candidates found, dropped by filter (with counts per reason), kept, total `actualCost`.

## Cost and stop rules

- Never enrich a candidate twice. Keep a map of domain to result for the whole session.
- Two failures on a step: stop that step and report it.
- Report `actualCost` per step and in total.

## When not to use this skill

- The user already has a list and wants to filter it by tools they use: `gtm-tech-stack-targeting`.
- One company: `gtm-account-research-brief`.
