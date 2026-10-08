---
name: gtm-tech-stack-targeting
description: Filter and segment an account list by the technology each company runs. Use when the user says "which of these companies use HubSpot", "find accounts running Shopify", "segment my list by CRM", "who uses a competitor's product", "tech stack targeting" or "displacement list for Intercom users". Detects each website's stack through looot, from $0.01 per account.
---

# Tech-stack targeting

Selling a HubSpot add-on to Salesforce shops wastes the whole sequence. This skill reads the tech stack of every account on a list, tags each one by the technologies the user cares about, and returns segments: uses the competitor, uses a complement, uses neither.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- The accounts: a list of domains.
- The technologies to look for, grouped as the user thinks of them: competitors (`Intercom`, `Drift`), complements (`Segment`), disqualifiers (`Salesforce` when the user only fits HubSpot shops).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `company.technographics` | `domain` | from $0.01 per result (6 providers) |
| 1 (fallback) | `company.tech-stack` | `domain` | from $0.065 per call |
| 2 (optional) | `web.scrape.html` | `url` of the homepage, to confirm a tag in the page source | from $0.00044 per call |

Expected total for 100 domains: about $1 with `company.technographics`.

## Quote before you run

1. Domains × $0.01 (or `costPerSuccessUsd` from `search_catalog`). Per-result pricing can bill more than one result per domain on some providers, so read the price formula in `inspect` and quote the upper bound.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.05). `company.tech-stack` costs more than that on its own, so it only runs when the user raises the cap.

## Run it

```tool
run {"endpointId": "job:company.technographics", "input": {"domain": "acme.com"}, "idempotencyKey": "tech-<list>-acme.com", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.05}}
```

Match technology names case-insensitively and by family (`HubSpot CRM`, `HubSpot Marketing Hub` and `HubSpot` all count as HubSpot). A technology missing from the result means "not detected", not "not used": back-end tools such as a CRM often leave no trace on the website. Say this in the output.

## Output format

```csv
domain,segment,competitors_found,complements_found,disqualifiers_found,all_detected,provider
acme.com,displace,Intercom,Segment,,"Intercom;Segment;Google Analytics;Cloudflare",<provider>
```

Segments: `displace` (a competitor found), `expand` (complement found, no competitor), `disqualified`, `unknown` (nothing relevant detected). Then counts per segment and the total `actualCost`.

## Cost and stop rules

- One run per domain. Dedupe domains first (strip `www.` and paths).
- Two failures in a row on different domains: stop and report.

## When not to use this skill

- One company's stack: `looot-enrich-a-company` or `gtm-account-research-brief`.
