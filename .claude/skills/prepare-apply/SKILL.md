---
name: prepare-apply
description: Produce a CV tailored to a specific job the user selected from /scan. Use when the user
  runs /prepare-apply <job#> or asks to tailor their CV / prepare an application for a job.
---

# Prepare Apply (demonstration)

> **Demo skill.** Generates an illustrative tailored CV from the saved profile and the selected job.
> A real version would rewrite the user's actual CV content, not a template.

## Steps

1. **Require a profile.** If `profiles/profile.md` doesn't exist, ask the user to run `/setup-profile`
   first, then stop.

2. **Identify the job.** Use the job number passed by the command; look it up in `data/mock-jobs.json`.
   If no number was given, ask the user which job (from `/scan`) they mean.

3. **Tailor (demo).** Read `profiles/profile.md` and the selected job. Produce a short CV that:
   - Leads with the profile's name and the job's title.
   - Highlights the profile skills that overlap the job's requirements.
   - Notes (as a placeholder) where real experience bullet points would be rewritten to match the role.

4. **Save** the result to `output/cv-<job#>.md` and tell the user where it is.

## Notes for a real implementation
- Start from the user's real CV and reorder / rephrase bullets to match the job's keywords.
- Flag skill gaps and suggest additions.
- Export to PDF / DOCX in addition to Markdown.
