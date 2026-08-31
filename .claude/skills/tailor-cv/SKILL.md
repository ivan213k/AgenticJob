---
name: tailor-cv
description: Adjust the user's baseline CV JSON (profiles/cv/cv.json) for one specific job and produce
  a change log of every adjustment. Used internally by prepare-apply after the job is resolved — not a
  standalone user-facing command.
---

# Tailor CV

Takes the baseline CV and one job, returns a job-optimized copy of the CV **in the same schema** plus
a change log the caller shows to the user before rendering. Works purely on JSON — never touch PDFs
here; rendering is the caller's job (`cv-renderer` MCP).

## Inputs

- `profiles/cv/cv.json` — the baseline CV in the renderer schema (see setup-profile). Never modify
  this file; it is the immutable baseline every tailoring starts from.
- The resolved job record: `title`, `company`, `location`, `description`, `requirements`, and — when
  it came from a scan — `summary` and `fitRationale` from `data/last-scan.json`.
- `profiles/profile.json` — for target titles and the user's own skill emphasis.

## Output

1. A tailored CV object, **valid against the same renderer schema** (same required fields, ISO
   `yyyy-MM-dd` dates, `endDate: null` for a current role). The caller writes it to the per-job folder.
2. A change log covering **every** difference from the baseline — nothing not listed may differ; it's
   the contract the user approves against. Keep it short and precise, formatted by change type (this
   is what the caller shows the user verbatim, so build it in this shape, not as prose):
   - **Rewritten text** (`summary`, `headline`, a `highlights` bullet, `projectDescription`) — a
     `diff`-fenced block, one `-`/`+` pair per changed field, no surrounding context lines:
     ```diff
     - .NET Software Engineer with over 4 years of experience with .NET Platform.
     + Senior .NET Engineer with 4+ years building microservice platforms on Azure.
     ```
   - **Reorders** (`skills`, `experience[].technologies`, bullet order) — one line, not a diff:
     `Skills: moved Docker, Kubernetes (AKS), Helm Charts to front`.
   - **Drops** — one line: `Dropped bullet: "…" (irrelevant to this role)`.
   - **Skill-gap notes** — one line each, only for genuinely unmet requirements (see the alias and
     requirement-reading rules): `Gap: job wants Kubernetes — not in your CV`, 
     `Bonus items not in your CV: React, Android`.
   - Group by section (headline/summary, per-role, skills) with a short heading per group; omit empty
     groups entirely rather than saying "no changes here".

## Rules

1. **Never fabricate.** Only reorder, rephrase, emphasize, or trim real content. `company`, `role`,
   dates, `education`, `languages`, and `certifications` are immutable. Never add a skill, technology,
   or achievement that isn't in the baseline — even if the job asks for it. Skill gaps are reported in
   the change log as notes ("job wants Kubernetes — not in your CV"), not papered over.

2. **Filter the job's `requirements` first.** Provider requirement arrays are noisy — they mix real
   skills ("ASP.NET Core", "Docker") with benefits and conditions ("Firmenfahrrad", "Weihnachtsgeld",
   "Festanstellung", degree names, languages). Extract the actual skills/technologies/keywords from
   `requirements` + `description` and tailor against those only.

3. **Normalize skill names to the posting's vocabulary — truthfully.** The same skill often goes by
   different names ("AKS" vs "Kubernetes", "GHA" vs "GitHub Actions", "Postgres" vs "PostgreSQL");
   recruiters and ATS filters search for the posting's exact term. Where the baseline holds a skill
   under an alias or abbreviation, rename it to the posting's term while keeping the specific variant
   in parentheses — e.g. "AKS" → "Kubernetes (AKS)" — and log it as a rewrite. This is
   canonicalization of a real skill, never an addition: if the baseline skill is genuinely narrower
   than the posting's term, it stays as-is. Apply the same alias check **before reporting a gap** — a
   skill present under another name is a rename, not a gap.

4. **Read requirements as the posting means them before reporting gaps.** Three cases that are not
   flat gaps:
   - **Alternatives lists** — "MongoDB, PostgreSQL, SQL Server, or similar" is one requirement
     satisfied by *any* listed (or genuinely similar) option. If the baseline has SQL Server, the
     requirement is met: no gap note at all, and never list the sibling options as missing.
   - **Nice-to-haves** — items the posting marks as bonus/plus/nice-to-have go in a separate line
     from hard requirements (`Bonus items not in your CV: …`), so the user can tell which gaps
     actually matter for applying.

5. **Typical adjustments** (each one logged):
   - `headline` — align with the job's title where the baseline honestly supports it (e.g. ".NET
     Software Engineer" → "Senior .NET Engineer" for a senior .NET posting; never claim a seniority
     or specialty the baseline doesn't back).
   - `summary` — rewrite to foreground the overlap between the baseline and this job; keep it factual
     and fluff-free.
   - `skills` — reorder so the job-matching skills lead; optionally drop clearly irrelevant trailing
     ones (logged).
   - `experience[].highlights` — rephrase bullets to use the job's terminology **where truthful**
     (same achievement, the job's vocabulary); reorder so the most relevant bullets lead; optionally
     drop a clearly irrelevant bullet (logged). Never inflate numbers or scope.
   - `experience[].technologies` — reorder so overlapping technologies lead.
   - `projectName` / `projectDescription` — minor rephrasing toward the job's domain where truthful.
   - `projects` — reorder so the most relevant standalone projects lead; optionally drop a clearly
     irrelevant one (logged). Same truthfulness rules as `experience`: no invented technologies or
     scope.

6. **Keep the language of the baseline CV** (don't translate the CV to the posting's language unless
   the user asks).

7. Return the tailored object and the change log to the caller — this skill does not write files, ask
   the user anything, or render PDFs itself.
