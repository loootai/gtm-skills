---
name: gtm-job-change-check
description: Check which past champions, customers or contacts changed jobs, and get their new work email. Use when the user says "which of my champions changed jobs", "check this contact list for job changes", "track former customers who moved companies", "who left their company" or "job change alerts for my CRM export". Reads each LinkedIn profile through looot (from $0.0012 each), then finds the new email for movers only.
---

# Job change check

People who bought from you before buy again at their next company. This skill takes a list of contacts with the company you last knew them at, reads each current LinkedIn profile, flags who moved, and finds a verified work email at the new company for the movers only.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- A list (CSV or pasted) with at least: name, last known company or domain, LinkedIn URL. Rows without a LinkedIn URL go through step 1b.
- Whether to find emails for movers (default yes).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `linkedin.person.profile` | `linkedin_url` | from $0.0012 per call |
| 1b | `people.enrich` (rows with no LinkedIn URL) | `first_name`, `last_name`, `company` or `domain` | from $0.0029 per call |
| 2 | `company.enrich` (movers only) | the new company name, to get its `domain` | from $0.0019 per call |
| 3 | `people.email.find` (movers only) | `first_name`, `last_name`, new `domain` | from $0.0036 per call |
| 4 | `people.email.verify` (movers only) | `email` | from $0.00145 per call |

Single-call job-change feeds also exist in the catalog: `people.job-change` (from $0.065 per call) and `people.job.changes` (from $0.108 per call). They cost 50 to 90 times more than one profile read, so use them only when the user wants a market-wide feed, not a check of their own list.

Expected total for 200 contacts with 10% movers: 200 × $0.0012 + 20 × ($0.0019 + $0.0036 + $0.00145) ≈ $0.38.

## Quote before you run

1. Count rows with and without a LinkedIn URL, multiply by the step 1 and 1b prices. Add an estimate for movers at 10% of rows times steps 2 to 4.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap every run with `fallback.maxCostUsd` (0.02 covers steps 1 to 4).

## Run it

```tool
run {"endpointId": "job:linkedin.person.profile", "input": {"linkedin_url": "https://www.linkedin.com/in/janedoe"}, "idempotencyKey": "jc-<list>-<row>-profile", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

Decide "moved" from the current position in the profile (the experience entry with no end date):

- Current company differs from the last known company, after normalising case, suffixes (Inc, Ltd, GmbH) and known rebrands: `moved`.
- Same company, new title: `promoted` (worth a note, no email needed).
- Same company and title: `stayed`.
- Profile not found or private: `unknown`. Do not guess.

Then for each `moved` row run steps 2 to 4, keyed `jc-<list>-<row>-find` and `jc-<list>-<row>-verify`.

## Output format

```csv
name,linkedin_url,last_known_company,status,current_company,current_title,started,new_domain,new_email,email_status
Jane Doe,https://www.linkedin.com/in/janedoe,Acme,moved,Example Org,VP Sales,2026-08,example.org,jane@example.org,valid
```

Then a summary: rows checked, moved, promoted, stayed, unknown, emails found and verified, total `actualCost`. List the movers first, newest start date on top.

## Cost and stop rules

- Steps 2 to 4 run only for `moved` rows.
- Keep `catch_all` and `risky` emails flagged, never mark them valid.
- Two failures in a row on step 1 with different profiles: stop and report, the provider may be down.

## When not to use this skill

- Finding new contacts at an account: `gtm-buying-committee-map`.
- A single email lookup: `looot-find-professional-emails` in `loootai/looot-skills`.
