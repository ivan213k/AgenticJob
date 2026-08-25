---
description: Generate a cover letter for a job into its application folder
argument-hint: <job# | job-url>
---

Invoke the **cover-letter** skill for `$ARGUMENTS` — either a job number from `/scan` or a job
posting URL. It writes `cover-letter.txt` into that job's folder under `output/` (reusing the folder
from `/prepare-apply` if one exists). If no profile or baseline CV exists, ask the user to run
`/setup-profile` first.
