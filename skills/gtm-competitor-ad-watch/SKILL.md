---
name: gtm-competitor-ad-watch
description: Track the ads your competitors run on Meta and Google, and what is new since last time. Use when the user says "what ads are my competitors running", "competitor ad watch", "new Facebook ads from these brands", "which competitor ads have run longest" or "ad angles my competitors test". Reads the Meta Ad Library and Google ads transparency data through looot, about $0.002 to $0.006 per competitor, and keeps state in a JSON file.
---

# Competitor ad watch

An ad that keeps running for months is an ad that converts. This skill reads each competitor's active ads, records when each ad started, flags new ads since the last check and the long runners, and groups the copy by angle so the user sees what competitors are testing.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- Competitors: brand names and domains (up to 20).
- Country (default `US`) and platforms (Meta by default, Google optional).
- Where state lives (default `ad-watch.json`).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `ads.meta.search` | the brand name or page, country | from $0.00188 per call |
| 2 (optional) | Google ads transparency: search the catalog for `google ads transparency` and use the job id it returns | the domain | check `inspect` before quoting |

Expected total: about $0.002 per competitor per check on Meta; 10 competitors weekly ≈ $0.02.

## Quote before you run

1. Competitors × platforms × the prices above. For step 2, quote only after `inspect`.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.02).

## Run it

Inspect once: some providers take a page id, others a search term; the paging field also differs.

```tool
run {"endpointId": "job:ads.meta.search", "input": {"query": "Acme", "country": "US"}, "idempotencyKey": "ads-2026-W41-acme", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

Keep only ads whose page name or link domain matches the competitor. For each ad record: ad id, start date, days running, headline, body, call to action, landing URL. Compare ad ids with the state file. Group copy into angles (price, speed, a named rival, social proof, a feature, an offer) by the main claim of the headline.

## Output format

```markdown
# Ad watch, week 2026-W41 (US, Meta), 6 competitors, cost $0.012

| Competitor | Active ads | New since last check | Longest running (days) | Top angle |
|---|---|---|---|---|
| Acme | 23 | 4 | 141 | price |

## New ads
- Acme, started 2026-10-04: "Half the price of <rival>" -> https://acme.com/compare (angle: price)

## Long runners (90+ days)
- Acme, 141 days: "Set up in 5 minutes" (angle: speed)
```

## Cost and stop rules

- One page of ads per competitor unless the user asks for more.
- Two failures on a competitor: list it under errors.

## When not to use this skill

- One brand's Meta ads, one time: `looot-meta-ads-library` in `loootai/looot-skills`.
