---
name: gtm-personalized-first-lines
description: Write a first line for each lead that cites a real, recent fact about the person or their company. Use when the user says "personalize the opener for these leads", "write first lines from their LinkedIn", "icebreakers for my list", "personalized intro line per prospect" or "research each lead and write a hook". Reads recent posts, company news and the website through looot, about $0.003 to $0.006 per lead, and never invents a fact.
---

# Personalized first lines

Generic openers ("I came across your profile") get ignored. This skill reads one to three real sources per lead, picks the most recent specific fact, and writes one sentence that cites it. Each line comes with the source URL so the user can check it before sending.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- The leads: name, company domain, LinkedIn URL when known.
- What the user sells, in one sentence, and the tone (plain, no flattery by default).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price. Stop at the first step that gives a usable fact.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `linkedin.person.posts` | `linkedin_url` | from $0.0015 per call |
| 2 | `company.news` | `domain` or `company` | from $0.000033 per call or $0.00025 per result |
| 3 | `web.scrape.markdown` | `url` of the company blog or homepage | from $0.0002 per call |

Expected total: about $0.003 per lead, $0.006 when all three steps run. 200 leads ≈ $0.60 to $1.20.

## Quote before you run

1. Leads × $0.006 as the upper bound.
2. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.02).

## Run it

```tool
run {"endpointId": "job:linkedin.person.posts", "input": {"linkedin_url": "https://www.linkedin.com/in/janedoe"}, "idempotencyKey": "fl-<list>-janedoe-posts", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

A usable fact is specific, from the last 90 days, and about the person's work or the company: a post they wrote, a launch, a hire, a funding round, a new market. Not usable: reshares with no comment, birthdays, generic quotes.

Write the line:

- One sentence, under 25 words.
- Names the fact, with the date when it helps ("Saw your post on Tuesday about cutting SDR ramp time to 6 weeks").
- Connects to what the user sells in at most a few words, or not at all.
- No praise words (amazing, impressive, love), no questions, no emojis.
- No usable fact: return `NO_FACT` and leave the line empty. Never make one up.

## Output format

```csv
name,company,first_line,fact,fact_date,source_type,source_url
Jane Doe,Acme,"Saw your post on Tuesday about cutting SDR ramp time to 6 weeks.","post on SDR ramp time",2026-10-06,linkedin_post,https://www.linkedin.com/posts/...
Sam Lee,Globex,,,,,NO_FACT
```

Summary: leads, lines written, NO_FACT count, total `actualCost`.

## Cost and stop rules

- Stop at the first source with a usable fact.
- Two failed reads for a lead: mark NO_FACT, move on.

## When not to use this skill

- A full account brief: `gtm-account-research-brief`.
