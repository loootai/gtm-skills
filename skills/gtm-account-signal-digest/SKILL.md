---
name: gtm-account-signal-digest
description: "Weekly digest of buying signals across a watchlist of target accounts: news, funding, hiring and website changes since the last run. Use when the user says \"what changed at my target accounts this week\", \"signal digest for my watchlist\", \"monitor these 30 accounts\", \"any news on my top accounts\" or \"weekly account alerts\". Runs news, job and page checks per account through looot, about $0.002 to $0.004 per account per week, and keeps state in a JSON file."
---

# Account signal digest

Paid signal tools charge a seat to tell you that a target account raised money or started hiring. This skill does the same check on demand for the user's own watchlist: it looks for new news, new matching job posts and a changed pricing or careers page per account, compares with the last run, and writes only what is new.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- The watchlist: domains with the company name, 5 to 200 accounts.
- Role words that count as a hiring signal (`SDR`, `RevOps`, `data engineer`).
- Pages to watch per account (default `/pricing` and `/careers` when they exist).
- Where state lives (default `signal-digest.json` in the working folder).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `news.search` | `query` = `"<company name>"` | from $0.00099 per call |
| 2 | `jobs.search` | company name plus role words | from $0.0005 per call |
| 3 | `web.scrape.markdown` | `url` of each watched page | from $0.0002 per call |

Expected total: about $0.002 per account with one watched page, $0.004 with two pages and a fallback. 100 accounts weekly ≈ $0.20 to $0.40 a week.

## Quote before you run

1. Accounts × (1 news + 1 jobs + watched pages) × the prices above.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes. A scheduled weekly run can reuse the yes for the same watchlist and the same quote; re-quote when the list grows.
3. Cap each run with `fallback.maxCostUsd` (0.02).

## Run it

Use the ISO week in idempotency keys (`sig-2026-W41-acme.com-news`), so a re-run in the same week replays for free.

```tool
run {"endpointId": "job:news.search", "input": {"query": "\"Acme Inc\""}, "idempotencyKey": "sig-2026-W41-acme.com-news", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

```tool
run {"endpointId": "job:web.scrape.markdown", "input": {"url": "https://acme.com/pricing"}, "idempotencyKey": "sig-2026-W41-acme.com-pricing", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

Compare with the state file:

- News: an article URL not seen before and published since the last run. Tag it `funding`, `exec_change`, `launch`, `partnership`, `layoffs` or `other` from the headline.
- Jobs: a posting URL not seen before whose title matches the role words.
- Pages: hash the markdown after removing dates and numbers that change every load; a different hash is a change. Quote up to 3 changed lines.

Save URLs and hashes, never full page text, in the state file.

## Output format

```markdown
# Signal digest, week 2026-W41, 42 accounts, cost $0.11

## Accounts with new signals (7)
### Acme (acme.com)
- funding: "Acme raises $20M Series B" (2026-10-06, https://...)
- hiring: 2 new SDR roles (https://..., https://...)
- pricing page changed: "Team plan $49 -> $59 per seat"

## No change (35)
acme2.com, ...

## Errors (0)
```

State file shape: `{"lastRun": "2026-10-08", "accounts": {"acme.com": {"news": ["<url>"], "jobs": ["<url>"], "pages": {"https://acme.com/pricing": "<sha256>"}}}}`.

## Cost and stop rules

- One news search and one jobs search per account per run.
- The first run has nothing to compare against: report it as a baseline, not as a list of signals.
- Two failures on an account: list it under Errors.

## When not to use this skill

- A deep one-off look at one account: `gtm-account-research-brief`. Funding across a whole segment: `gtm-funding-signal-scan`.
