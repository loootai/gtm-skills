---
name: gtm-local-prospect-list
description: Build a prospect list of local businesses with a verified contact email. Use when the user says "list of dentists in Lyon with emails", "find 100 gyms in Austin to pitch", "local business leads for my agency", "restaurants in Lisbon without a website", "prospect list from Google Maps". Searches Google Maps, reads each website for a contact email, verifies it through looot, about $0.003 to $0.006 per business.
---

# Local prospect list

Agencies and local-service sellers need lists like "every physio clinic in Manchester with a website and a reachable email". This skill searches Google Maps, filters on rating and website, reads each business's site for a public contact email, and verifies the email before it goes on the list.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- Business type and place: `dentist`, `Lyon, France`. Several places are fine.
- How many businesses (default 50).
- Filters: minimum rating, minimum review count, has a website (default yes), or the opposite (no website, a lead for a web agency).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `maps.local.search` | `query`, `location` | from $0.001 per call or $0.000539 per result |
| 2 | `web.scrape.markdown` | `url` of the website, then `/contact` or the contact link found on it | from $0.0002 per call |
| 3 | `people.email.verify` | each `email` found | from $0.00145 per call |
| 3b (optional) | `people.domain.search` | `domain`, when no email is on the site | from $0.0089 per call |

Expected total: about $0.003 per business with a site email, $0.012 when step 3b runs. 100 businesses: $0.30 to $1.20.

## Quote before you run

1. Searches (1 per place per page of about 20 results) + 2 page reads per business + 1 verify per email found. Add step 3b only if the user asks.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.02, or 0.05 when step 3b runs).

## Run it

```tool
run {"endpointId": "job:maps.local.search", "input": {"query": "dentist", "location": "Lyon, France"}, "idempotencyKey": "local-dentist-lyon-p1", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

Filter before step 2: drop permanently closed places, places under the rating or review filters, and duplicates (same website or same phone). Then for each website:

```tool
run {"endpointId": "job:web.scrape.markdown", "input": {"url": "https://example.com"}, "idempotencyKey": "local-dentist-lyon-example.com-home", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

Pull emails with a plain pattern match on the markdown, prefer an address on the business's own domain, drop image file names and obvious placeholders. If none is on the homepage, read the contact page once. Then verify each address.

## Output format

```csv
name,category,address,city,phone,website,rating,reviews,email,email_status,email_source,maps_url
Cabinet Dentaire Example,Dentist,"12 rue X",Lyon,+33 4 00 00 00 00,https://example.com,4.8,212,contact@example.com,valid,https://example.com/contact,https://maps.google.com/...
```

Summary: found on Maps, kept after filters, with an email, verified valid, total `actualCost`. `catch_all` and `risky` stay on the list with their status; `invalid` addresses are dropped.

## Cost and stop rules

- Only public business addresses from the business's own pages. Do not guess personal emails.
- Two failed page reads on a site: mark `no_email`, move on.
- Respect the user's local rules on cold email (for example, consent rules in the EU).

## When not to use this skill

- Just the list of places: `looot-local-business-search`. Reviews of one place: `looot-google-maps-reviews`. Both in `loootai/looot-skills`.
