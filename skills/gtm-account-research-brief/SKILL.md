---
name: gtm-account-research-brief
description: Write a one-page research brief on a target account before outreach or a call. Use when the user says "research acme.com before my call", "account brief for these 5 companies", "what should I know about this prospect", "prep me for a meeting with Acme" or "pre-call research". Pulls firmographics, LinkedIn page, tech stack, open roles, recent news and the homepage through looot, about $0.02 to $0.05 per account.
---

# Account research brief

Turns a company domain into a one-page brief a rep can read in two minutes: what the company does, size and funding, what it runs on, who it is hiring, what changed recently, and three angles for the first message. Every fact carries its source.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching the catalog and inspecting are free. Runs are paid from the prepaid balance.

## Ask for these inputs

- One or more company domains (`acme.com`). A company name works for step 1, but a domain is more reliable.
- What the user sells, in one sentence. The angles in the brief depend on it.
- Optional: the person they are meeting, with a LinkedIn URL.

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08 (`GET https://api.looot.ai/v1/catalog/overview?depth=jobs`). The provider looot routes to can cost more. `inspect` shows the live price and the most a run can hold.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `company.enrich` | `domain` | from $0.0019 per call |
| 2 | `linkedin.company.profile` | `linkedin_url` from step 1, or `domain` | from $0.0015 per call |
| 3 | `company.technographics` | `domain` | from $0.01 per result |
| 4 | `jobs.search` | the company name, plus role words if useful | from $0.0005 per call |
| 5 | `news.search` | `"<company name>"` as `query` | from $0.00099 per call |
| 6 | `web.scrape.markdown` | `url` of the homepage, then `/pricing` if it exists | from $0.0002 per call |
| 7 (optional) | `linkedin.person.profile` | the person's `linkedin_url` | from $0.0012 per call |

Expected total: about $0.02 per account with the cheapest providers, up to about $0.05 when a step falls back. Steps 3 to 6 do not depend on each other and can run in parallel.

## Quote before you run

1. Count the accounts and the steps you will run. Multiply by the per-step prices above (or by `costPerSuccessUsd` from `search_catalog`).
2. Tell the user the total and the current `balance`, then wait for a yes. Above 100 runs in one go, always wait for an explicit yes.
3. Put a cap on every run with `fallback.maxCostUsd`, for example `0.05`. A provider priced above the cap is skipped and nothing is charged for it.

## Run it

Run each step as a job with fallback and a fresh idempotency key per account and step (`<domain>-enrich`, `<domain>-news`):

```tool
run {"endpointId": "job:company.enrich", "input": {"domain": "acme.com"}, "idempotencyKey": "brief-acme.com-enrich", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.05}}
```

```tool
run {"endpointId": "job:news.search", "input": {"query": "\"Acme Inc\""}, "idempotencyKey": "brief-acme.com-news", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

For `jobs.search` and `company.technographics`, call `inspect` once first: input names differ between providers, and `jobInputs` in a `search_catalog` answer lists the ones each job takes.

Read `status` and `error` of every run, not only the HTTP status. A `miss` is a normal answer: write "not found" in the brief, do not guess.

## Output format

One markdown block per account:

```markdown
# Acme Inc (acme.com)
Sources: company.enrich via <provider>, news.search via <provider>, ... | cost $0.023

## Snapshot
- What they do: <one sentence from the homepage, quoted where possible>
- Size: <employees> | HQ: <city, country> | Founded: <year>
- Funding: <last round, amount, date> or "not found"

## Stack
<up to 10 technologies, grouped: CRM, marketing, data, cloud>

## Hiring
<count of open roles, the 3 most relevant titles with links>

## Recent news (last 90 days)
- <date> <headline> (<source link>)

## Angles for the first message
1. <angle tied to one fact above, with the fact named>
2. ...
3. ...
```

Rules for the angles: each one points at a fact in the brief. No invented pain points, no flattery lines.

## Cost and stop rules

- Never loop a failing call. Two failures on the same step: skip the step, mark it "not found" and say so.
- Report the total `actualCost` at the end, per account and overall.
- Do not run step 3 (per-result pricing) on more than 20 accounts without a fresh quote.

## When not to use this skill

- One fact only (headcount, email format): use the single-job skills in `loootai/looot-skills`.
- A list of many accounts to qualify against an ICP: use `gtm-icp-lookalike-list` or `gtm-tech-stack-targeting`.
