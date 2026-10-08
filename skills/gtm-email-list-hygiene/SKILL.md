---
name: gtm-email-list-hygiene
description: Clean an email list before a campaign so it does not bounce. Use when the user says "clean this email list", "verify these 2,000 emails before I send", "remove bad emails from my CSV", "check bounce risk", "email list hygiene" or "dedupe and verify my leads". Dedupes and checks syntax locally for free, drops disposable domains, then verifies each address through looot from $0.00145 each.
---

# Email list hygiene

A bounce rate above 2 to 3 percent hurts a sending domain. This skill takes a list, removes what can be removed for free (duplicates, bad syntax, role and disposable addresses if the user wants), verifies the rest, and returns the list split into send, review and drop.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- The list: a CSV with an email column (keep every other column as is).
- Whether to drop role addresses (`info@`, `sales@`, `support@`), default: keep but flag.
- What to do with catch-all domains, default: put them in review.

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 0 | none, local | lowercase, trim, dedupe, syntax check (one `@`, a dot in the domain, no spaces) | free |
| 1 | `people.email.disposable-check` | `email` (one per domain is enough) | $0 per call |
| 2 | `people.email.verify` | `email` | from $0.00145 per call |
| 2 (bulk) | `people.email.verify.batch` | a list of emails | from $0.004 per result; use for 1,000+ rows only after comparing prices |

Expected total: 1,000 unique valid-syntax addresses × $0.00145 ≈ $1.45.

## Quote before you run

1. Do step 0 first and report what it removed. Count what is left: that is the number of paid verifications.
2. Remaining × $0.00145 (or `costPerSuccessUsd` from `search_catalog`). Tell the user the total and the `balance`; wait for a yes. This skill almost always crosses 100 runs, so the yes must be explicit.
3. Cap each run with `fallback.maxCostUsd` (0.01).

## Run it

```tool
run {"endpointId": "job:people.email.verify", "input": {"email": "jane@example.com"}, "idempotencyKey": "hyg-<list>-jane@example.com", "fallback": {"maxAttempts": 2, "maxCostUsd": 0.01}}
```

Run 5 to 10 at a time. Read `normalized` when present (`status`, `deliverable`), else the provider's `result`. Map every answer to one of: `valid`, `catch_all`, `risky`, `invalid`, `unknown`.

## Output format

Three CSVs with the user's original columns plus `email_status` and `reason`:

- `send.csv`: `valid`
- `review.csv`: `catch_all`, `risky`, `unknown`, role addresses
- `drop.csv`: `invalid`, bad syntax, duplicates, disposable

Summary table:

```markdown
| Step | Rows | Note |
|---|---|---|
| Input | 2,140 | |
| Duplicates removed | 96 | free |
| Bad syntax | 12 | free |
| Disposable | 18 | free |
| Verified | 2,014 | $2.92 |
| Send | 1,702 | 84.5% |
| Review | 241 | catch-all 198, risky 31, unknown 12 |
| Drop | 71 | invalid |
```

## Cost and stop rules

- Never verify the same address twice in a session.
- An `unknown` answer is not a reason to retry on another provider more than once.
- Two failures in a row with different addresses: pause and report.

## When not to use this skill

- One address: `looot-email-verification` in `loootai/looot-skills`.
