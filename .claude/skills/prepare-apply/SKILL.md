---
name: prepare-apply
description: Build a real per-job application package — tailored CV (JSON + rendered PDF) and cover
  letter — into output/<job-folder>/. Use when the user runs /prepare-apply <job# | job-url> or asks
  to tailor their CV / prepare an application for a job.
---

# Prepare Apply

Real skill. Produces one folder per job under `output/` containing the fluff-free job summary, the
tailored CV (JSON + PDF rendered by the `cv-renderer` MCP server), and a cover letter. The LLM works
only with CV JSON; PDF layout is entirely the renderer's job.

## Steps

1. **Preconditions.** Require both `profiles/profile.json` and `profiles/cv/cv.json` (the baseline CV
   in the renderer schema). If either is missing, ask the user to run `/setup-profile` — it also
   handles adding just `cv.json` to an existing profile — then stop.

2. **Resolve the job.** The argument is either a job number from `/scan` or a job posting URL; if
   neither was given, ask which the user means.

   *Job number:*
   1. Look up the number in `data/last-scan.json` for its `provider` and `id`.
   2. Prefer the freshest copy: call the `get_{provider}_job(id)` MCP tool (cache-only, works within
      the job-sources cache TTL, ~7 days). If it returns empty, fall back to the `Job` fields already
      stored in `data/last-scan.json`.
   3. If the number isn't in `data/last-scan.json` at all, tell the user to run `/scan` first, then
      stop. Never fall back to `data/mock-jobs.json` — it's unused demo data.
   4. Give a 2-line recap (title, company, why it fit) and proceed — the user already chose this job
      in `/scan`.

   *URL:*
   1. Fetch the page (WebFetch). Job boards often block automated fetching — if the fetch is blocked,
      empty, or clearly not the posting, say so honestly and ask the user to paste the posting text
      instead. Never invent posting content.
   2. Summarize the posting **fluff-free**: title, company, location, seniority, the actual
      requirements (skills/technologies — skip benefits and marketing), and any notable conditions
      (remote policy, language, salary if stated).
   3. Show the summary, then ask whether to proceed with preparing the CV **via the AskUserQuestion
      tool** (Yes / No options) — a popup, not a free-text question. Stop cleanly on No.

3. **Create the per-job folder** `output/<company-slug>-<role-slug>/` — kebab-case, each part
   truncated to a few words so the path stays sane; if the folder exists for a *different* job,
   suffix `-2`. Create it **once**, with a single `mkdir -p` folded into writing `<folder>/job.md`;
   every later step assumes it exists — no repeated "ensure folder exists" checks, they're pure
   noise in the user's transcript. `job.md` holds the fluff-free summary plus the source
   (`provider` + `id` and link, or the URL), so the folder is self-describing and `/cover-letter`
   can find it later.

4. **Tailor.** Invoke the **tailor-cv** skill with the baseline `profiles/cv/cv.json`, the resolved
   job, and `profiles/profile.json`. Write the tailored result to `<folder>/cv.json`. The baseline
   file is never modified.

5. **Showcase & approve.** Show tailor-cv's change log to the user **verbatim** — it's already
   formatted for precision (diff blocks for rewrites, one-liners for reorders/drops/gaps). Don't
   restate it as prose or add extra narration. Then ask for approval **via the AskUserQuestion tool**
   (popup, not free text) with options like "Approve — render the PDF" and "Request edits" (the
   built-in "Other" lets the user type specific changes directly). On edit requests, apply them to
   `<folder>/cv.json`, re-show the updated change log in the same format, and ask again — repeat
   until approved. Do **not** render before approval — the download URL expires, so a premature
   render is wasted.

6. **Render & download.** Call `render_cv` on the `cv-renderer` MCP server with the tailored JSON.
   The response contains a download URL with an `expiresAt` — download it **immediately**; never
   save the URL for later. Save as `<folder>/<Firstname>_<Lastname>_CV.pdf` (from the CV's
   `fullName`, e.g. `Ivan_Zakharuk_CV.pdf`) — recruiter-facing convention: the filename must
   identify the candidate, underscores instead of spaces for ATS/email safety, no company name.
   Use exactly this command shape (a permission rule allowlists this prefix, so any deviation —
   different flags, flag order, or an absolute output path — triggers a needless prompt):

   ```
   curl -sS -o output/<folder>/<Firstname>_<Lastname>_CV.pdf "<url>"
   ```
   Verify the file is non-empty and starts with the `%PDF` magic bytes. If the server is unreachable
   or errors, report that honestly (the tailored `cv.json` is still saved) — don't substitute a
   fake PDF.

7. **Cover letter — on demand.** Ask **via the AskUserQuestion tool** (Yes / No popup) whether to
   also draft a cover letter. On Yes, invoke the **cover-letter** skill for the same job, passing the
   resolved job record and the folder so it skips re-resolution → `<folder>/cover-letter.txt`. On No,
   skip it — the user can run `/cover-letter` for this job later.

8. **Report.** List the folder contents with paths: `job.md`, `cv.json`,
   `<Firstname>_<Lastname>_CV.pdf`, and `cover-letter.txt` if it was drafted.
