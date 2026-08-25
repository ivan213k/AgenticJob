# AgenticJob — AI Context

## What this repo is
AgenticJob is a **proof-of-concept** for an agentic job-search-and-apply assistant driven entirely
through Claude Code slash commands and skills. There is no traditional application to run — the
"product" is the set of commands and skills in `.claude/`.

## ⚠️ Scope guardrail
The whole pipeline is now **real** — explicitly requested and built out step by step:
- `/setup-profile` stores the actual profile and CV, including a baseline `profiles/cv/cv.json`.
- `/scan` fetches real postings from the `job-sources` MCP server.
- `/prepare-apply` tailors the real CV JSON (via the `tailor-cv` skill) and renders a PDF through
  the real `cv-renderer` MCP server; `/cover-letter` drafts from real CV achievements.

**Do not build further integrations beyond this (ATS/apply submission, auth, new providers, etc.)
unless the user explicitly asks.** No skill may ever fall back to mock data (`data/mock-jobs.json`)
or fabricate CV content — if a server is unreachable, report that honestly.

## Workflow
1. `/setup-profile` — collects the user's actual profile (name, target job title, preferences) plus a
   **required CV upload**, and saves `profiles/profile.json` (+ rendered `profiles/profile.md`), the
   original CV, and the baseline `profiles/cv/cv.json` in the cv-renderer schema. Other commands
   should prompt the user to run this first if no profile / baseline CV exists.
2. `/scan [count]` — ask which job-source provider to use (via the real `job-sources` MCP server),
   fetch real postings, score them against the profile (`score-jobs` skill), and list the best
   `count` as short, fluff-free summaries. If the server is unreachable it reports that rather than
   substituting mock data.
3. `/prepare-apply <job# | job-url>` — build a per-job application package in
   `output/<company>-<role>/`: fluff-free `job.md`, tailored `cv.json` (via the `tailor-cv` skill,
   never fabricating content), `<Firstname>_<Lastname>_CV.pdf` rendered by the `cv-renderer` MCP
   server (approval-gated: the
   change log vs the baseline CV is shown and approved before rendering), and — only if the user
   says yes when asked — `cover-letter.txt`. A URL argument is fetched and summarized first, then
   the user confirms before tailoring.
4. `/cover-letter <job# | job-url>` — write/refresh just the cover letter into that job's folder.

## Conventions
- **Skills are the building block.** Each real behavior lives in a skill under `.claude/skills/<name>/SKILL.md`.
- **Commands are thin wrappers.** Each `.claude/commands/<name>.md` just invokes its matching skill and
  passes arguments through.
- Naming is **kebab-case** for both commands and skills.

## Key paths
- `.claude/commands/` — slash-command entry points (thin).
- `.claude/skills/` — the logic: setup-profile, scan-jobs, score-jobs, prepare-apply, tailor-cv,
  cover-letter.
- `.mcp.json` — **real** MCP servers (streamable-HTTP): `job-sources` (search/resolve postings) and
  `cv-renderer` (JsonToCvApi — `render_cv(cv)` renders CV JSON to PDF, returns an expiring download
  URL that must be downloaded immediately).
- `data/last-scan.json` — cached results of the last `/scan` (gitignored); maps display job numbers
  to `(provider, id)` so `/prepare-apply` and `/cover-letter` can resolve `<job#>`.
- `data/mock-jobs.json` — old fake job postings; kept for reference only, no longer read by any skill.
- `profiles/` — user profile + CV (gitignored); `profiles/cv/cv.json` is the immutable baseline CV
  every tailoring starts from.
- `output/<company>-<role>/` — one folder per job application: `job.md`, `cv.json`,
  `<Firstname>_<Lastname>_CV.pdf`, `cover-letter.txt` (gitignored).
