<p align="center"><img src="assets/hero.png" alt="gtm-skills: 15 GTM workflows as agent skills, built on looot" width="100%"></p>

# gtm-skills

[![License](https://img.shields.io/github/license/loootai/gtm-skills)](LICENSE) [![Release](https://img.shields.io/github/v/release/loootai/gtm-skills)](https://github.com/loootai/gtm-skills/releases) [![Docs](https://img.shields.io/badge/docs-docs.looot.ai-12A06A)](https://docs.looot.ai)

15 agent skills for go-to-market work: account research, lookalike lists, job-change and hiring signals, tech-stack segments, competitor SERP and ad watches, local prospect lists, inbound enrichment and list hygiene. Each skill is a multi-step workflow with the exact data jobs it runs, the price of every step, a quote-before-run rule and a fixed output format.

The data comes from [looot](https://looot.ai): one key and one prepaid balance for 2,500+ data API endpoints from 90+ providers. The agent sees the price before it runs and pays per call. A failed call costs nothing. No seat, no subscription, top up from $5.

## Install

```bash
npx skills add loootai/gtm-skills
claude mcp add --transport http looot https://api.looot.ai/mcp
```

The first line uses the [skills CLI](https://github.com/vercel-labs/skills) and installs into Claude Code, Cursor, Codex and the other agents it supports. Add `--list` to see the skills first, or `--skill gtm-job-change-check` for one. The second line connects the looot MCP server (browser sign-in, no key to copy). Other clients: https://docs.looot.ai.

For single data jobs (one email, one SERP, one company), install [loootai/looot-skills](https://github.com/loootai/looot-skills) too. This repo holds the workflows that chain them.

## Skills

Prices are the cheapest live provider in the public looot catalog on 2026-10-08 (`GET https://api.looot.ai/v1/catalog/overview?depth=jobs`, no key needed). Each skill tells the agent to quote the total and wait for a yes before it spends.

| Skill | What it does | Typical cost |
|---|---|---|
| `gtm-account-research-brief` | One-page brief on an account: firmographics, stack, hiring, news, three angles | ~$0.02 to $0.05 per account |
| `gtm-icp-lookalike-list` | Ranked account list from 1 to 5 seed customers | under $1 per 50 accounts |
| `gtm-job-change-check` | Which past champions moved, with a verified email at the new company | ~$0.38 per 200 contacts |
| `gtm-hiring-signal-scan` | Accounts hiring for the roles that predict a purchase | ~$0.05 to $0.10 per 100 accounts |
| `gtm-tech-stack-targeting` | Segment a list into displace, expand, disqualified by detected tech | ~$1 per 100 domains |
| `gtm-competitor-serp-watch` | Google top 10 and AI Mode share of voice vs competitors, with changes | ~$0.003 per keyword |
| `gtm-competitor-ad-watch` | Competitors' active Meta ads, new ads and long runners, grouped by angle | ~$0.002 per competitor |
| `gtm-local-prospect-list` | Google Maps businesses with a contact email from their site, verified | ~$0.003 to $0.012 per business |
| `gtm-inbound-lead-enrichment` | Sign-up emails to person, company and ICP score; free-mail skipped | ~$0.0055 per business email |
| `gtm-email-list-hygiene` | Dedupe, syntax, disposable and verify, split into send, review, drop | ~$1.45 per 1,000 addresses |
| `gtm-funding-signal-scan` | Recently funded companies in a segment, or rounds on your list | ~$0.40 per 30 companies |
| `gtm-buying-committee-map` | Economic buyer, champion, user and evaluator per account, with emails | ~$0.025 per account |
| `gtm-post-engagers-to-leads` | People who reacted to or commented on a LinkedIn post, filtered by ICP | ~$0.015 per 300 engagers |
| `gtm-personalized-first-lines` | One opener per lead that cites a real, recent, linked fact | ~$0.003 to $0.006 per lead |
| `gtm-account-signal-digest` | Weekly news, hiring and page changes across a watchlist | ~$0.20 to $0.40 per 100 accounts a week |

## How every skill is built

- `SKILL.md` with `name` (equal to the folder) and a `description` that lists the phrases that should trigger it.
- A steps table: looot job id, input, price on 2026-10-08. Job ids are run as `job:<id>` with `fallback`, so looot tries the next provider when one misses, inside one cost cap.
- A quote-before-run rule: count the runs, multiply by the prices, tell the user the total and the balance, wait for a yes, and an explicit yes above 100 runs. Every run carries `fallback.maxCostUsd`.
- Stable idempotency keys per row and step, so a retry never pays twice.
- A fixed output format (CSV columns or a markdown template) and stop rules: two failures, stop and report.
- Paid feeds that cost 10 to 100 times more than the cheap route are named, with the price, and only used when the user asks.

Prices change. The agent reads the live price with `inspect` before the first run, and the live price wins over the table.

## Run without MCP

Every step is one HTTP call. With an agent token from https://looot.ai/settings in `LOOOT_TOKEN`:

```bash
curl -s -X POST "https://api.looot.ai/v1/runs?wait=30" \
  -H "Authorization: Bearer $LOOOT_TOKEN" -H "Content-Type: application/json" \
  -d '{"endpointId":"job:company.enrich","input":{"domain":"stripe.com"},"idempotencyKey":"brief-stripe.com-enrich","fallback":{"maxAttempts":3,"maxCostUsd":0.05}}'
```

## See also

- [looot-skills](https://github.com/loootai/looot-skills): single data jobs as skills
- [looot-tables](https://github.com/loootai/looot-tables): open-source enrichment tables on Next.js and Supabase
- [awesome-gtm](https://github.com/loootai/awesome-gtm): open-source GTM tools

## License

MIT. See [LICENSE](LICENSE).
