---
name: gtm-post-engagers-to-leads
description: Turn the people who reacted to or commented on a LinkedIn post into a lead list filtered by your ICP. Use when the user says "who engaged with this LinkedIn post", "get leads from a competitor's post", "scrape commenters on this post", "warm leads from post engagement" or "people who liked this post that match my ICP". Reads reactions and comments through looot (from $0.0015 per page), filters by headline, then enriches only the matches.
---

# Post engagers to leads

People who comment on a post about a problem are telling you they have it. This skill reads the reactions and comments on one or more LinkedIn posts (the user's own, a competitor's, an influencer's), keeps the people whose headline matches the ICP, and optionally finds their work email.

Needs looot connected (MCP server `https://api.looot.ai/mcp`, or the `looot` skill from `npx skills add loootai/looot-skills`). Searching and inspecting are free; runs are paid from the prepaid balance.

## Ask for these inputs

- One or more LinkedIn post URLs.
- The ICP as headline words: titles (`founder`, `head of growth`, `RevOps`) and words to exclude (`student`, `recruiter`, `looking for`).
- Whether to find emails for the matches (default no; it costs more and needs the company domain).

## Steps and prices

Prices are the cheapest live provider in the public looot catalog on 2026-10-08. `inspect` shows the live price.

| # | Job id | Input | Price on 2026-10-08 |
|---|---|---|---|
| 1 | `linkedin.post.reactions` | the post URL | from $0.0015 per call (one page) |
| 2 | `linkedin.post.comments` | the post URL | from $0.0015 per call (one page) |
| 3 (matches only) | `linkedin.person.profile` | `linkedin_url` | from $0.0012 per call |
| 4 (optional) | `people.email.find` then `people.email.verify` | `first_name`, `last_name`, `domain` | from $0.0036 + $0.00145 |

Expected total: a post with 300 reactions and 40 comments, about 10 pages ≈ $0.015; 30 matches with profiles ≈ $0.04; with emails ≈ $0.19.

## Quote before you run

1. Read the post's reaction and comment counts if the user knows them; estimate pages at about 50 people per page (inspect shows the provider's page size).
2. Pages × $0.0015, plus the expected matches × steps 3 and 4. Tell the user the total and the `balance`; wait for a yes. More than 100 runs: an explicit yes.
3. Cap each run with `fallback.maxCostUsd` (0.02).

## Run it

Search first, then inspect the endpoint you will use: the input name for the post (URL, URN or id) and the paging field differ by provider, and `jobInputs` in the search answer lists them.

```tool
search_catalog {"query": "reactions on a linkedin post"}
```

```tool
run {"endpointId": "job:linkedin.post.comments", "input": {"url": "https://www.linkedin.com/posts/..."}, "idempotencyKey": "eng-<post-id>-comments-p1", "fallback": {"maxAttempts": 3, "maxCostUsd": 0.02}}
```

Merge reactions and comments by profile URL. Commenters rank above reactors. Match the headline against the ICP words, drop excludes, then read full profiles only for the matches to get the current company.

## Output format

```csv
name,headline,linkedin_url,engagement,comment_text,current_company,current_title,company_domain,email,email_status,post_url
Jane Doe,Head of Growth at Acme,https://www.linkedin.com/in/janedoe,comment,"We tried 3 tools for this...",Acme,Head of Growth,acme.com,,,https://...
```

Summary: engagers read, matches, emails found, total `actualCost`. Quote the comment text as is; it is the opener's best material.

## Cost and stop rules

- Profiles and emails only for matches.
- Stop paging when a page returns no new people.
- Use the list for one-to-one outreach. Do not mass-message everyone who reacted.

## When not to use this skill

- A person's own posts: `linkedin.person.posts` through the base `looot` skill.
