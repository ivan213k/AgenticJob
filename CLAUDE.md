# AgenticJob — AI Context

## What this repo is
AgenticJob is a **proof-of-concept** for an agentic job-search-and-apply assistant driven entirely
through Claude Code slash commands and skills. There is no traditional application to run — the
"product" is the set of commands and skills in `.claude/`.

## ⚠️ Demonstration-only
Everything here is a **skeleton for demonstration purposes**. Skills contain illustrative,
prose-level instructions and use **mock data** — they do NOT talk to real job boards, parse real
CVs, or call real APIs. **Do not build real integrations (LinkedIn scraping, live MCP servers,
real CV tailoring, auth, etc.) unless the user explicitly asks.**

## Workflow
1. `/setup-profile` — the one **real** step in this POC. Collects the user's actual profile (name,
   target job title, preferences) plus a **required CV upload**, and saves structured data to
   `profiles/profile.json` (+ a rendered `profiles/profile.md` and the CV under `profiles/cv/`). Other
   commands should prompt the user to run this first if no profile exists.
2. `/scan [count]` — list summarized jobs (default 10) from `data/mock-jobs.json`, stripping employer
   "fluff" into short summaries.
3. `/prepare-apply <job#>` — produce a demo tailored CV for a selected job into `output/`.
4. `/cover-letter <job#>` — produce a demo cover letter for a selected job into `output/`.

## Conventions
- **Skills are the building block.** Each real behavior lives in a skill under `.claude/skills/<name>/SKILL.md`.
- **Commands are thin wrappers.** Each `.claude/commands/<name>.md` just invokes its matching skill and
  passes arguments through.
- Naming is **kebab-case** for both commands and skills.

## Key paths
- `.claude/commands/` — slash-command entry points (thin).
- `.claude/skills/` — the actual demo logic (setup-profile, scan-jobs, prepare-apply, cover-letter).
- `data/mock-jobs.json` — fake job postings used by `/scan` (intentionally verbose so summarization has work to do).
- `.mcp.json` — placeholder `job-sources` MCP server config (non-functional stub, marks the real integration point).
- `profiles/` — where the user profile is written (gitignored).
- `output/` — where tailored CVs / cover letters are written (gitignored).
