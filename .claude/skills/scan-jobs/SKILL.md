---
name: scan-jobs
description: List available jobs as short, fluff-free summaries so the user can decide what to apply
  to. Use when the user runs /scan or asks to see / search for jobs. Honors an optional count (default 10).
---

# Scan Jobs (demonstration)

> **Demo skill.** Reads mock jobs from `data/mock-jobs.json`. A real version would call live job-source
> APIs / an MCP server instead (see below).

## Steps

1. **Require a profile.** If `profiles/profile.md` doesn't exist, ask the user to run `/setup-profile`
   first, then stop.

2. **Determine count.** Use the count passed by the command; default to **10** if none was given.

3. **Fetch jobs.** For this demo, read `data/mock-jobs.json` (all entries — the mock data has no
   filtering, so pull everything).

4. **Rank by relevance to the profile.** Compare each job's title, requirements, and description
   against the profile's target job title, skills, and preferences (from `profiles/profile.md`).
   Favor jobs whose title/stack matches the target role and whose requirements overlap most with the
   profile's skills; use location/remote preference as a tiebreaker. Sort best-match first, then take
   the top `count`.

5. **Strip the fluff.** Each mock posting has verbose marketing copy. Condense each into a **1–2 line
   summary** that keeps only what matters to a candidate: what the role actually does and standout
   requirements. Drop slogans, "rockstar/ninja" language, and boilerplate perks.

6. **Present a numbered list**, including each job's source platform, publish date, and link, e.g.:

   ```
   1. Senior Backend Engineer — Acme Corp (Remote)
      Build and scale payment APIs in Go; needs 5y backend + distributed systems.
      Posted 2026-06-12 via LinkedIn — https://www.linkedin.com/jobs/view/3618348156

   2. ...
   ```

   Tell the user they can run `/prepare-apply <#>` or `/cover-letter <#>` for any listed job.

## Real integration point (placeholder)
Replace step 3 with calls to real job sources — e.g. `GET /api/linkedinjobs?filter=<value>`,
`GET /api/otherServiceJobs?filter=<value>`, or the `job-sources` MCP server declared in `.mcp.json`.
Those APIs would accept filters derived from the saved profile (target title, location, skills), but
even filtered results should still be ranked (step 4) before truncating to `count` — filtering narrows
the pool, it doesn't guarantee the top `count` are the best matches.
