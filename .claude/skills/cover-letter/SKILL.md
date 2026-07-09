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

2. **Identify the job.** Use the job number passed by the command; look it up in `data/mock-jobs.json`.
   If no number was given, ask the user which job (from `/scan`) they mean.

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
