---
name: scan-jobs
description: List available jobs as short, fluff-free summaries so the user can decide what to apply
  to. Use when the user runs /scan or asks to see / search for jobs. Honors an optional count (default 10).
---

# Scan Jobs

Queries the real `job-sources` MCP server (see `.mcp.json`) for one provider at a time, scores the
results against the user's profile via the **score-jobs** skill, and presents a ranked, numbered
list.

## Steps

1. **Require a profile.** If `profiles/profile.md` doesn't exist, ask the user to run `/setup-profile`
   first, then stop.

2. **Determine count.** Use the count passed by the command; default to **10** if none was given.
   This is the number of jobs to *display* — step 6 will fetch more than this.

3. **Ask which provider.** List the MCP tools exposed by the `job-sources` server and find every
   tool named `search_{provider}_jobs` — each one is an available provider (don't hardcode a
   provider list; a newly added provider should show up automatically). Present the provider names
   via `AskUserQuestion` as a single-select choice. Only call the chosen provider's tools for this
   scan — **never call multiple providers and merge results.**

4. **Derive the search request** from `profiles/profile.json`:
   - `search`: a plain keyword from `target_job_titles` (this is a plain case-insensitive
     substring match now, not regex — no escaping needed).
   - `countryCode`: the ISO 3166-1 alpha-2 code inferred from the profile's `location` country
     (e.g. "Leipzig, Saxony, Germany" → `DE`). If it can't be inferred confidently, ask the user.
   - `locations`: the profile's city/region, plus `"Remote"` when `remote_preference` is
     remote/hybrid/flexible.
   - `preferredSkills`: the profile's `skills` list. Use `preferredSkills` (at-least-one-match),
     not `mustHaveSkills` (all-must-match) — the goal is a candidate pool for scoring, not a hard
     filter that could zero it out.
   - `searchAliases`: fallback search terms for the same role, **strongest match first** (e.g. for
     `search: "C# Developer"`, aliases like `[".NET Developer", "C# Entwickler"]`) — only used by
     the server when `search` alone returns fewer than `take` results, so it's cheap to always
     populate a few from the profile's other `target_job_titles` / obvious synonyms.
   - `take`: request **more than `count`** (e.g. `count * 3`, capped around 30–50) so `score-jobs`
     in step 6 has a real pool to choose from — raw provider filtering is coarse; profile-fit
     judgment happens in step 6, not here.

5. **Show the search request and let the user adjust it.** Before calling the provider, print the
   derived parameters plainly (provider, `search`, `searchAliases`, `locations`, `countryCode`,
   `preferredSkills`), e.g.:

   ```
   Searching LinkedIn for:
     Title:      C# Developer  (aliases: .NET Developer, C# Entwickler)
     Location:   Leipzig, Remote  (DE)
     Skills:     C#, .NET, SQL, Azure
   ```

   Ask if this looks right or if they want to adjust anything (different location, broader/narrower
   title, different skills, etc.) before searching — a plain "looks good" / enter is enough to
   proceed. Apply any adjustments the user gives directly to the query arguments (they override the
   profile-derived defaults for this scan only; don't write them back to `profiles/profile.json`
   unless the user says to). Re-show the updated parameters if they changed something, then proceed
   once confirmed.

6. **Fetch a candidate pool (overfetch).** Call the chosen provider's `search_{provider}_jobs` MCP
   tool with the confirmed arguments from steps 4–5.

   - **Zero or very few results:** tell the user, suggest loosening filters (fewer
     `preferredSkills`, broader `locations`), and offer to retry relaxed.
   - **Fewer survive scoring than `count` (step 7 says so):** if the tool's `totalCount` indicates
     more results exist beyond this page, retry with `skip` advanced to the next page before
     giving up.
   - **MCP call fails / server unreachable:** tell the user clearly that the real job-sources
     server is unreachable (include the error if known), and **stop — do not fall back to mock
     data.**

7. **Score and rank.** Invoke the **score-jobs** skill with the fetched candidate pool, the
   profile, and `count`. Use its output (ranked jobs with condensed summaries and fit rationale)
   directly for the next step.

8. **Present a numbered list**, including each job's source platform, publish date, and link, e.g.:

   ```
   1. Senior Backend Engineer — Acme Corp (Remote)
      Build and scale payment APIs in Go; needs 5y backend + distributed systems.
      Posted 2026-06-12 via LinkedIn — https://www.linkedin.com/jobs/view/3618348156

   2. ...
   ```

   Tell the user they can run `/prepare-apply <#>` or `/cover-letter <#>` for any listed job.

9. **Cache the results.** Write the displayed list to `data/last-scan.json`, keyed by display
   number, including: the `provider`, the job's `id` **exactly as returned by the MCP tool** (treat
   it as an opaque token — don't parse or cast it, the API is expected to switch it from integer to
   string), the full `Job` fields, and the `score-jobs` summary/rationale. `/prepare-apply` and
   `/cover-letter` resolve `<job#>` from this file.

## Data source
Real data only, via the `job-sources` MCP server. If it's unreachable there is no mock fallback —
`data/mock-jobs.json` is kept in the repo for reference but is no longer read by this skill.
