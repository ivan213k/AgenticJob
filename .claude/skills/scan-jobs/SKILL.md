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

3. **Fetch jobs.** For this demo, read `data/mock-jobs.json`. Take up to `count` entries.

4. **Strip the fluff.** Each mock posting has verbose marketing copy. Condense each into a **1–2 line
   summary** that keeps only what matters to a candidate: what the role actually does and standout
   requirements. Drop slogans, "rockstar/ninja" language, and boilerplate perks.

5. **Present a numbered list**, e.g.:

   ```
   1. Senior Backend Engineer — Acme Corp (Remote)
      Build and scale payment APIs in Go; needs 5y backend + distributed systems.

   2. ...
   ```

   Tell the user they can run `/prepare-apply <#>` or `/cover-letter <#>` for any listed job.

## Real integration point (placeholder)
Replace step 3 with calls to real job sources — e.g. `GET /api/linkedinjobs?filter=<value>`,
`GET /api/otherServiceJobs?filter=<value>`, or the `job-sources` MCP server declared in `.mcp.json`.
Filters would be derived from the saved profile (target title, location, skills).
