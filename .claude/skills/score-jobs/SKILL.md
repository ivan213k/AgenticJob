---
name: score-jobs
description: Rank a pool of candidate jobs against the user's profile and condense each into a
  fluff-free summary with a fit rationale. Used internally by scan-jobs after fetching candidates
  from a job-sources provider; also available to prepare-apply/cover-letter for fit rationale. Not
  a standalone user-facing command.
---

# Score Jobs

Given a pool of candidate jobs and the user's profile, pick and rank the best matches — this is
where profile-fit judgment happens, separate from whatever coarse filtering a job source already
applied (keyword/skill/location filters narrow the pool, they don't guarantee the survivors are the
*best* matches).

## Inputs
- `profiles/profile.json` — the user's profile (`target_job_titles`, `skills`, `location`,
  `remote_preference`, `seniority`, etc.).
- A list of candidate `Job` objects (`id, title, company, location, description, requirements,
  link, sourcingPlatform, datePublished` — from a real job-sources search or any other source; this
  skill doesn't care where they came from).
- A target `count` — how many ranked results the caller wants.

## Steps

1. **Score each candidate.** Compare title, `requirements`, and `description` against the
   profile's `target_job_titles` and `skills`. Favor jobs whose title/stack matches the target
   role and whose requirements overlap most with the profile's skills. Use location match /
   `remote_preference` as a tiebreaker, not a hard filter.

2. **Condense.** For each candidate, reduce the (often marketing-heavy) `description` into a
   **1–2 line, fluff-free summary**: what the role actually does, and the standout requirements.
   Drop slogans, "rockstar/ninja" language, and boilerplate perks.

3. **Write a fit rationale.** One short clause per job on *why* it's a good match for this
   profile specifically (e.g. "direct .NET/Azure stack overlap, senior-level"). Keep it terse —
   this is meant to be reusable as-is by `prepare-apply`/`cover-letter` when they want to explain
   why they tailored content the way they did.

4. **Sort and truncate.** Best match first, return the top `count`.

5. **Report shortfall honestly.** If fewer than `count` candidates are worth returning (e.g. the
   pool was too small or too weak a match), say so explicitly rather than padding the list — let
   the caller decide whether to fetch a bigger/different pool and retry.

## Output
For each returned job: the original `Job` fields, the condensed summary, and the fit rationale,
in ranked order.
