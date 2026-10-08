---
name: gtm-buying-committee-map
description: Map the buying committee at a target account and get verified emails for the people who matter. Use when the user says "who should I contact at acme.com", "map the buying committee", "find the decision makers at these accounts", "multi-thread this deal", "who signs off on a CRM at Acme". Searches people by role at the domain, picks one to three per role, finds and verifies their work emails through looot, about $0.02 to $0.05 per account.
---

# Buying committee map

Deals with one contact stall. This skill finds the people behind each role in a purchase (economic buyer, champion, user, technical evaluator) at an account, and returns them with verified emails and a short note on why each one is on the map.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- Account domains (1 to 50).
- What the user sells, so the roles fit. Example for a sales tool: economic buyer = VP Sales or CRO, champion = Sales Ops or RevOps lead, user = SDR or AE manager, technical = Sales Systems or IT.
- How many people per role (default 1, max 3).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `people.search` | `domain` plus a `query` per role (`"VP Sales"`) | $0 per call, from $0.00036 per result |
| 2 | `people.email.find` | `first_name`, `last_name`, `domain`, `linkedin_url` when known | from $0.0036 per call |
| 3 | `people.email.verify` | `email` | from $0.00145 per call |
| 4 (optional) | `linkedin.person.profile` | `linkedin_url`, to confirm the current title | from $0.0012 per call |

Expected total per account with 4 roles × 1 person: about $0.005 search + 4 × $0.005 emails ≈ $0.025; with profile checks ≈ $0.03.

## Quote before you run

1. Accounts × roles × people per role × (steps 2 + 3), plus the searches.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.03).

## Run it

```tool
run {"endpointId": "job:people.search", "input": {"domain": "acme.com", "query": "VP Sales"}, "idempotencyKey": "bcm-acme.com-economic", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.03}}
```

Pick per role: current title matches the role words, most senior first for the economic buyer, most specific title first for the champion. Drop people whose current company is not the account. Then find and verify emails only for the people you picked:

```tool
run {"endpointId": "job:people.email.find", "input": {"first_name": "Jane", "last_name": "Doe", "domain": "acme.com"}, "idempotencyKey": "bcm-acme.com-jane-doe-find", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.03}}
```

## Output format

```markdown
## Example Inc (example.com), cost $0.027
| Role | Name | Title | LinkedIn | Email | Status | Why |
|---|---|---|---|---|---|---|
| Economic buyer | Jane Doe | VP Sales | https://... | jane@example.com | valid | owns the sales budget |
| Champion | Sam Lee | RevOps Manager | https://... | sam@example.com | catch_all | runs the tools the product replaces |
| User | | | | | | not found |
```

Plus one CSV with all rows for import. Empty roles stay in the table as "not found".

## Cost and stop rules

- Never find emails for people you did not pick.
- A role with no match: do not widen to unrelated titles without asking.
- Two failures in a row on step 1: stop and report.

## When not to use this skill

- One person's email: `looot-find-professional-emails`. Everyone at a domain: `looot-enrich-a-company` or `people.domain.search`.
