---
name: setup-profile
description: Collect the user's real job-search profile — contact info, target role, preferences, and
  a required CV upload — and save it under profiles/. Use when the user runs /setup-profile, asks to
  set up or update their profile, or when another job-search step needs a profile that doesn't exist yet.
---

# Setup Profile

> **Real skill.** This is the one part of the AgenticJob POC that collects and stores actual user data
> (see `CLAUDE.md`). `/scan`, `/prepare-apply`, and `/cover-letter` still run on mock job data — only
> profile collection is real here.

## Storage layout

```
profiles/
  profile.json        # structured source of truth — other skills read this
  profile.md            # human-readable render of profile.json
  cv/
    original.<ext>      # exact copy of the uploaded CV (pdf or txt/md)
    extracted.md          # plain-text extraction: contact, experience, skills, education
```

`profiles/*` is gitignored — none of this leaves the user's machine via git.

## Steps

1. **Check for an existing profile.** If `profiles/profile.json` exists, show a short summary (name,
   target title, whether a CV is on file) and ask whether to keep it, update individual fields, or
   redo it from scratch. If the user only wants to update a few fields, skip straight to those and
   re-render `profile.md` at the end — don't re-ask everything.

2. **Require the CV before anything else.** This is mandatory — do not save a profile without one.
   Ask the user for the **local file path** to their CV. Storing the CV requires a byte-for-byte copy
   of the original file (see step 3), which needs an actual filesystem path — a chat attachment's
   content alone isn't enough, so ask for the path directly even if the user has already shared/attached
   the file. Read that path with the Read tool.

   **Never search the filesystem to locate the CV.** Do not run scans like `find /` or `find ~` to
   guess at it — a wide search on a real machine can turn up unrelated sensitive files (SSH/VPN keys,
   credentials, other documents) that just happen to match a name fragment, which is both a privacy
   risk and unreliable at picking the right file. If the user doesn't know the exact path, only search
   inside folders they explicitly name (e.g. "check my Downloads") with a shallow, targeted glob —
   never an unscoped root/home search.
   - **Accepted formats: PDF or plain text/Markdown.** If they provide a `.docx` or other format the
     Read tool can't parse, ask them to export/save it as PDF (or paste the text directly) — don't
     attempt to parse it anyway.
   - Once you have a valid path, read it with the Read tool to confirm it's parseable before moving on.
   - If the user has no CV available right now, stop and tell them to run `/setup-profile` again once
     they have one — don't create a partial profile without a CV.

3. **Store the CV.**
   - Copy the original file byte-for-byte into `profiles/cv/original.<ext>` (matching the source
     extension). Use a filesystem copy (e.g. `cp`), never rewrite it through a text-writing tool —
     that will corrupt a PDF.
   - From your Read of the CV, write a plain-text/Markdown extraction to `profiles/cv/extracted.md`
     covering: contact details found in the CV, work history (company, title, dates, bullet points as
     written), education, skills/technologies, certifications. This is what `prepare-apply` and
     `cover-letter` should read from later — don't summarize away real bullet points, keep them intact.

4. **Collect the rest of the profile.** Pre-fill suggestions from the CV extraction where you can
   (name, most recent title, skills, years of experience) and let the user confirm or correct each
   one rather than re-typing from scratch. Ask for:
   - **Full name** (required — suggest from CV)
   - **Target job title(s)** (required) — one or a short list, e.g. "Senior Backend Engineer"
   - Email, phone (optional — suggest from CV)
   - Location and remote/hybrid/onsite preference (optional)
   - Seniority / years of experience (optional — suggest from CV)
   - Key skills, comma-separated (optional — suggest from CV, let the user trim/extend)
   - Links: LinkedIn, portfolio, GitHub (optional)
   - Work authorization / visa status (optional)
   - Salary expectation, availability (optional)

   Only name, target job title, and CV are required — skip anything else the user doesn't provide,
   don't block on it.

5. **Save `profiles/profile.json`** as the source of truth:

   ```json
   {
     "name": "<name>",
     "email": "<email or null>",
     "phone": "<phone or null>",
     "location": "<location or null>",
     "remote_preference": "<remote|hybrid|onsite|flexible or null>",
     "target_job_titles": ["<title>", "..."],
     "seniority": "<seniority or null>",
     "years_experience": <number or null>,
     "skills": ["<skill>", "..."],
     "links": { "linkedin": "<url or null>", "portfolio": "<url or null>", "github": "<url or null>" },
     "work_authorization": "<value or null>",
     "salary_expectation": "<value or null>",
     "availability": "<value or null>",
     "cv": {
       "original_filename": "<name the user uploaded>",
       "original_path": "profiles/cv/original.<ext>",
       "extracted_path": "profiles/cv/extracted.md",
       "uploaded_at": "<ISO date>"
     },
     "updated_at": "<ISO date>"
   }
   ```

6. **Render `profiles/profile.md`** from that JSON — a human-readable view, e.g.:

   ```markdown
   # Profile: <name>

   - Target job title(s): <titles>
   - Location: <location or "—"> · <remote_preference or "—">
   - Seniority: <seniority or "—"> (<years_experience or "—"> yrs)
   - Key skills: <skills or "—">
   - Links: <linkedin>, <portfolio>, <github>
   - CV: `profiles/cv/original.<ext>` (extracted: `profiles/cv/extracted.md`)
   ```

7. **Confirm** to the user that the profile and CV were saved, and that they can now run `/scan` to
   see jobs.
