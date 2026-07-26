---
name: cover-letter
description: Generate a cover letter for a specific job the user selected from /scan. Use when the user
  runs /cover-letter <job#> or asks to write / draft a cover letter for a job.
---

# Cover Letter (demonstration)

> **Demo skill.** Generates an illustrative cover letter from the saved profile and the selected job.

## Steps

1. **Require a profile.** If `profiles/profile.md` doesn't exist, ask the user to run `/setup-profile`
   first, then stop.

2. **Identify the job.** Use the job number passed by the command; if no number was given, ask the
   user which job (from `/scan`) they mean. Resolve the number to a job record:
   1. Look up the number in `data/last-scan.json` for its `provider` and `id`.
   2. Prefer the freshest copy: call the `get_{provider}_job(id)` MCP tool (works as long as that
      job was scanned within the job-sources cache TTL, ~7 days). If it returns empty (expired or
      never cached), fall back to the `Job` fields already stored in `data/last-scan.json`.
   3. If the number isn't in `data/last-scan.json` at all (no scan run yet, or out of range), tell
      the user to run `/scan` first, then stop. Do not fall back to `data/mock-jobs.json` — it's
      unused demo data, kept in the repo only for reference.

3. **Draft (demo).** Read `profiles/profile.md` and the selected job. Write a short, ~3-paragraph cover
   letter that:
   - Opens by naming the role and company.
   - Connects 2–3 of the profile's skills to the job's needs.
   - Closes with a call to action.

   Keep the tone genuine — avoid generic filler.

4. **Save** the result to `output/cover-letter-<job#>.md` and tell the user where it is.

## Notes for a real implementation
- Pull concrete achievements from the user's real CV/history for the middle paragraph.
- Offer tone variants (formal / conversational) and length options.
- Auto-fill the hiring manager / company address when available from the job source.
