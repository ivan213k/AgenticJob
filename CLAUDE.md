# AgenticJob — AI Context

## What this repo is
AgenticJob is a **proof-of-concept** for an agentic job-search-and-apply assistant driven entirely
through Claude Code slash commands and skills. There is no traditional application to run — the
"product" is the set of commands and skills in `.claude/`.

## ⚠️ Demonstration-only (except /scan)
Most of this repo is a **skeleton for demonstration purposes**. Skills contain illustrative,
prose-level instructions and use **mock data** — they do NOT parse real CVs, tailor real CV
content, or call real ATS/apply APIs. **Do not build further real integrations (auth, real CV
rewriting, etc.) unless the user explicitly asks.**

The one exception: `/scan` is wired to a **real** job-sources MCP server (see `.mcp.json`) and
returns real job postings — this was explicitly requested and built out. `/prepare-apply` and
`/cover-letter` still generate demo/template output, but they resolve the job itself from a real
scan's results rather than mock data.

## Workflow
1. `/setup-profile` — a **real** step. Collects the user's actual profile (name,
   target job title, preferences) plus a **required CV upload**, and saves structured data to
   `profiles/profile.json` (+ a rendered `profiles/profile.md` and the CV under `profiles/cv/`). Other
   commands should prompt the user to run this first if no profile exists.
2. `/scan [count]` — ask which job-source provider to use (Indeed/Stepstone, via the real
   `job-sources` MCP server), fetch real postings, score them against the profile (`score-jobs`
   skill), and list the best `count` as short, fluff-free summaries. If the server is unreachable
   it reports that rather than substituting mock data.
3. `/prepare-apply <job#>` — produce a demo tailored CV for a selected job into `output/`.
4. `/cover-letter <job#>` — produce a demo cover letter for a selected job into `output/`.

## Conventions
- **Skills are the building block.** Each real behavior lives in a skill under `.claude/skills/<name>/SKILL.md`.
- **Commands are thin wrappers.** Each `.claude/commands/<name>.md` just invokes its matching skill and
  passes arguments through.
- Naming is **kebab-case** for both commands and skills.

## Key paths
- `.claude/commands/` — slash-command entry points (thin).
- `.claude/skills/` — the logic: setup-profile, scan-jobs, score-jobs, prepare-apply, cover-letter.
- `.mcp.json` — **real** `job-sources` MCP server config (streamable-HTTP), used by scan-jobs /
  prepare-apply / cover-letter to search and resolve real job postings.
- `data/last-scan.json` — cached results of the last `/scan` (gitignored); maps display job numbers
  to `(provider, id)` so `/prepare-apply` and `/cover-letter` can resolve `<job#>`.
- `data/mock-jobs.json` — old fake job postings; kept for reference only, no longer read by any skill.
- `profiles/` — where the user profile is written (gitignored).
- `output/` — where tailored CVs / cover letters are written (gitignored).
