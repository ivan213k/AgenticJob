---
name: setup-profile
description: Collect the user's real job-search profile — contact info, target role, preferences, and
  a required CV upload — and save it under profiles/. Use when the user runs /setup-profile, asks to
  set up or update their profile, or when another job-search step needs a profile that doesn't exist yet.
---

# Setup Profile

> **Real skill.** Collects and stores actual user data (see `CLAUDE.md`). The baseline
> `profiles/cv/cv.json` written here is what `/prepare-apply` tailors and renders to PDF, and what
> `/cover-letter` draws real achievements from.

## Storage layout

```
profiles/
  profile.json        # structured source of truth — other skills read this
  profile.md            # human-readable render of profile.json
  cv/
    original.<ext>      # exact copy of the uploaded CV (pdf or txt/md)
    cv.json               # structured extraction in the cv-renderer schema — the baseline CV
```

`profiles/*` is gitignored — none of this leaves the user's machine via git.

## Steps

1. **Check for an existing profile.** If `profiles/profile.json` exists, show a short summary (name,
   target title, whether a CV is on file) and ask whether to keep it, update individual fields, or
   redo it from scratch. If the user only wants to update a few fields, skip straight to those and
   re-render `profile.md` at the end — don't re-ask everything.

   **Migration:** if a profile exists but `profiles/cv/cv.json` is missing, offer to generate just it —
   from a legacy `profiles/cv/extracted.md` if present (delete `extracted.md` afterwards, it's
   superseded), otherwise from `original.<ext>` — following step 3's conversion rules, without redoing
   anything else. Update `profile.json`'s `cv` object accordingly.

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
   - From your Read of the CV, build the **baseline CV** `profiles/cv/cv.json` in the `cv-renderer`
     schema — this is what `prepare-apply` tailors and `cover-letter` reads later:

     ```json
     {
       "fullName": "…", "headline": "…", "summary": "…",
       "contact": { "email": "…", "location": "…", "phone": null, "linkedInUrl": null },
       "skills": ["…"],
       "experience": [{
         "role": "…", "company": "…",
         "startDate": "yyyy-MM-dd", "endDate": "yyyy-MM-dd or null",
         "highlights": ["…"], "technologies": ["…"],
         "projectName": null, "projectDescription": null
       }],
       "education": [{ "degree": "…", "institution": "…", "location": "…",
                       "startDate": "yyyy-MM-dd", "endDate": "yyyy-MM-dd" }],
       "languages": [{ "name": "…", "level": "…" }],
       "certifications": [{ "name": "…", "issuedDate": "yyyy-MM-dd", "url": null }]
     }
     ```

     Conversion rules:
     - Dates are ISO `yyyy-MM-dd`; a month-only date becomes the first of that month; a current role
       gets `endDate: null`.
     - `highlights` are the CV's bullet points **as written** — don't summarize away real bullets.
     - A per-role technologies list in the CV goes to `technologies`; per-role project prose goes to
       `projectName` / `projectDescription`.
     - `fullName`, `headline` (current title), `summary`, `skills`, `education`, `languages`,
       `certifications` map straight from the CV; `contact.email` and `contact.location` are required
       by the renderer — ask the user if the CV doesn't state them.
     - If the CV holds something the schema can't (e.g. publications), tell the user it won't appear
       in rendered PDFs — don't silently drop it.

     Show the resulting JSON to the user for confirmation before saving.

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
       "json_path": "profiles/cv/cv.json",
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
   - CV: `profiles/cv/original.<ext>` (baseline JSON: `profiles/cv/cv.json`)
   ```

7. **Confirm** to the user that the profile and CV were saved, and that they can now run `/scan` to
   see jobs.
