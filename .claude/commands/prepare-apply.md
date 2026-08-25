---
description: Prepare a full application package (tailored CV PDF + cover letter) for a job
argument-hint: <job# | job-url>
---

Invoke the **prepare-apply** skill for `$ARGUMENTS` — either a job number from `/scan` or a job
posting URL. It builds a per-job folder in `output/` with the job summary, tailored CV (JSON + PDF),
and cover letter. If no profile or baseline CV exists, ask the user to run `/setup-profile` first.
