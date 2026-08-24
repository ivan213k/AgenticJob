---
name: cover-letter
description: Write a real cover letter for a specific job into that job's output/ folder. Use when the
  user runs /cover-letter <job# | job-url> or asks to write / draft a cover letter for a job. Also
  invoked by prepare-apply as part of the full application package.
---

# Cover Letter

Real skill. Drafts a cover letter grounded in the user's actual CV and saves it into the same per-job
folder that `/prepare-apply` uses, so each job's application package lives in one place.

## Steps

1. **Preconditions.** Require `profiles/profile.json` and `profiles/cv/cv.json`. If either is
   missing, ask the user to run `/setup-profile` first, then stop.

2. **Resolve the job — unless the caller already did.** When invoked from **prepare-apply**, the
   resolved job record and target folder are passed in: skip straight to step 4. Otherwise the
   argument is a job number from `/scan` or a posting URL:

   *Job number:*
   1. Look up the number in `data/last-scan.json` for its `provider` and `id`.
   2. Prefer the freshest copy: call the `get_{provider}_job(id)` MCP tool (cache-only, ~7 day TTL).
      If it returns empty, fall back to the `Job` fields stored in `data/last-scan.json`.
   3. If the number isn't in `data/last-scan.json` at all, tell the user to run `/scan` first, then
      stop. Never fall back to `data/mock-jobs.json`.

   *URL:* fetch with WebFetch and summarize fluff-free; if the fetch is blocked or empty, ask the
   user to paste the posting text instead.

3. **Locate or create the job folder.** Check existing `output/*/job.md` files for one whose source
   (provider + id, or URL) matches this job and reuse that folder. Otherwise create
   `output/<company-slug>-<role-slug>/` and write `job.md` the same way prepare-apply does (fluff-free
   summary + source), so a later `/prepare-apply` reuses it too. Create the folder at most once (a
   single `mkdir -p` folded into writing `job.md`) — never re-check or re-create a folder that's
   already there, and when the caller passed the folder in, trust it.

4. **Draft.** Read `profiles/cv/cv.json` and the job. Write a ~3-paragraph letter that:
   - Opens by naming the role and company.
   - Uses 1–2 **concrete achievements** from the CV's `experience[].highlights` (real bullets, not
     generic skill claims) and connects them to the job's actual needs.
   - Closes with a call to action.

   Keep the tone genuine — no generic filler, no achievements or skills that aren't in the CV. Write
   in the language of the baseline CV unless the user asks otherwise. Offer tone (formal /
   conversational) and length variants if the user wants alternatives.

   **Plain text, not markdown.** This gets pasted into portal text boxes or email bodies, not
   rendered — no `#` headers, `**bold**`, or `-` bullets. Format it like an actual letter: salutation
   line, blank-line-separated paragraphs, sign-off with the candidate's name from `profiles/cv/cv.json`.

5. **Save** to `<folder>/cover-letter.txt` and tell the user where it is.
