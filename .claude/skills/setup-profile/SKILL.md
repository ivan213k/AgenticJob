---
name: setup-profile
description: Collect and save the user's job-search profile — name, target job title, and a few
  preferences. Use when the user runs /setup-profile, asks to set up or update their profile, or when
  another job-search step needs a profile that doesn't exist yet.
---

# Setup Profile (demonstration)

> **Demo skill.** This walks the user through a minimal profile and saves it as a Markdown file.
> A full version would also import/parse an existing CV, validate inputs, and store richer structured data.

## Steps

1. **Check for an existing profile.** If `profiles/profile.md` already exists, show a short summary and
   ask whether the user wants to keep it or update it. Otherwise continue.

2. **Ask the user for their basics.** Keep it short — for this POC, collect:
   - **Full name** (required)
   - **Target job title** (required) — e.g. "Senior Backend Engineer"
   - Location / remote preference *(optional)*
   - Seniority *(optional)*
   - Key skills, comma-separated *(optional)*
   - Path to an existing CV *(optional — not parsed in this demo)*

   Only name and target job title are required; skip the rest if the user doesn't provide them.

3. **Save the profile** to `profiles/profile.md` using this template:

   ```markdown
   # Profile: <name>

   - Target job title: <target job title>
   - Location: <location or "—">
   - Seniority: <seniority or "—">
   - Key skills: <skills or "—">
   - CV: <cv path or "—">
   ```

4. **Confirm** to the user that the profile was saved and that they can now run `/scan` to see jobs.

## Notes for a real implementation
- Parse the provided CV to auto-fill skills and experience.
- Store structured JSON alongside the Markdown for downstream steps.
- Support multiple named profiles / target roles.
