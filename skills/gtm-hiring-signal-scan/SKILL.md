---
name: gtm-hiring-signal-scan
description: Scan a list of target accounts for hiring signals that predict a purchase. Use when the user says "which of these companies are hiring SDRs", "find accounts hiring for RevOps", "hiring signals for my account list", "who is building a data team" or "companies posting jobs that mention Salesforce". Searches open roles per account through looot, from $0.0005 per account.
---

# Hiring signal scan

A company hiring three SDRs needs sales tools; a company hiring its first data engineer needs data tools. This skill searches open job postings for each account, keeps the roles that match the user's buying signals, and ranks accounts by signal strength.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- The accounts: domains or company names (a list or a CSV).
- The signal: role titles (`SDR`, `RevOps`, `Data Engineer`) and/or words in the job text (`Salesforce`, `HubSpot`, `Snowflake`).
- How fresh: postings from the last 30 days by default.

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `jobs.search` | company name plus role words as the query | from $0.0005 per call or $0.000145 per result |
| 1 (alt) | `jobs.company` | the company (domain or LinkedIn URL) | from $0.01 per call |
| 2 (optional) | `web.scrape.markdown` | `url` of a matching posting, to read the full text | from $0.0002 per call |

`signals.hiring` (from $0.108 per call) returns a ready-made hiring signal feed. Use it only when the user asks for a market-wide feed, not to check their own list.

Expected total for 100 accounts: about $0.05 to $0.10 with `jobs.search`, about $1 with `jobs.company`.

## Quote before you run

1. Accounts × the price of the job you picked, plus step 2 for up to 3 postings per matching account.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.02).

## Run it

Inspect `jobs.search` once: providers differ on whether they take a free `query`, a `company` field or a location. `jobInputs` in a `search_catalog` answer lists what each one takes.

```tool
run {"endpointId": "job:jobs.search", "input": {"query": "Acme SDR", "company": "Acme"}, "idempotencyKey": "hire-<list>-acme.com", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

Keep a posting only when the employer name matches the account (job boards mix in agencies and similarly named companies) and the title or text matches the signal. Score each account: 3 points per matching role posted in the window, +2 when the role is a first hire for that function ("first", "founding", "build the team"), +1 when the text names one of the signal tools.

## Output format

```csv
domain,company,score,matching_roles,newest_posting,example_title,example_url,signal_note
acme.com,Acme,8,2,2026-10-01,Senior SDR,https://...,"2 SDR roles in 30 days, text names HubSpot"
```

Then the accounts with no matching role, and the total `actualCost`.

## Cost and stop rules

- One search per account; do not page further unless the user asks.
- Two failures on the same account: mark it `error`, move on.
- Report `actualCost` at the end.

## When not to use this skill

- One company's open roles in full: `looot-job-postings` in `loootai/looot-skills`.
