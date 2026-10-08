---
name: gtm-inbound-lead-enrichment
description: Enrich and qualify inbound sign-ups or demo requests from just an email address. Use when the user says "enrich these sign-ups", "who are the people who requested a demo", "qualify my inbound leads", "score these form fills against our ICP", "route inbound leads" or "enrich a lead from an email". Enriches the person and company through looot and scores the lead, about $0.006 per business email; personal emails are not enriched.
---

# Inbound lead enrichment

A sign-up form gives you an email. This skill turns each one into a person, a company and a fit score, so the team calls the right leads first. Free-mail and disposable addresses are sorted out before any paid call.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- The leads: emails, optionally with name and the form fields they filled.
- The ICP in plain terms: industries, employee range, countries, titles that buy.
- Routing rules if any ("score 7+ goes to sales, the rest to nurture").

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 0 | none, local check | free-mail domain list (gmail.com, outlook.com, yahoo.com, icloud.com, proton.me...) | free |
| 1 | `people.email.disposable-check` | `email` | $0 per call |
| 2 | `people.enrich.by-email` | `email` | from $0.0036 per call |
| 3 | `company.enrich` | `domain` of the email | from $0.0019 per call |
| 4 (optional) | `people.email.verify` | `email`, when the form had no confirmation step | from $0.00145 per call |

Expected total: about $0.0055 per business email, $0.007 with verification. 500 sign-ups with 40% business emails: about $1.10 to $1.40.

## Quote before you run

1. Sort locally first: count business emails (paid steps) and free-mail or disposable ones (no paid steps).
2. Business emails × the step prices. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.02).

## Run it

```tool
run {"endpointId": "job:people.enrich.by-email", "input": {"email": "jane@example.com"}, "idempotencyKey": "inbound-<batch>-jane@example.com-person", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

```tool
run {"endpointId": "job:company.enrich", "input": {"domain": "example.com"}, "idempotencyKey": "inbound-<batch>-example.com-company", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

Enrich each company domain once, even when several leads share it. Score each lead out of 10: industry match 3, employee range 3, country 1, title matches a buying title 3. Missing data scores 0 for that part and is listed, not guessed.

## Output format

```csv
email,type,name,title,seniority,linkedin_url,company,domain,industry,employees,country,score,route,missing
jane@example.com,business,Jane Doe,VP Sales,vp,https://www.linkedin.com/in/janedoe,Example Inc,example.com,Software,240,US,9,sales,
joe@example.net,free_mail,,,,,,,,,,,nurture,"not enriched: personal email"
```

Then: totals per route, the top 10 leads with one line each on why they scored high, total `actualCost`.

## Cost and stop rules

- Never pay to enrich free-mail or disposable addresses.
- Two failures in a row on step 2 with different emails: stop and report.
- Personal data stays in the user's systems. Do not post it anywhere else.

## When not to use this skill

- Cleaning a list before a send: `gtm-email-list-hygiene`. One person: `looot-enrich-a-person` in `loootai/looot-skills`.
