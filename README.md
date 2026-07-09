# AgenticJob

Simplify the job search and application process with automation and agentic workflows, driven
through **Claude Code** slash commands and skills.

> **Status: proof-of-concept skeleton.** This repo is a *demonstration* of the intended workflow.
> The commands and skills use mock data and illustrative instructions — they do **not** connect to
> real job boards or generate real applications yet.

## The workflow

| Step | Command | What it does |
| ---- | ------- | ------------ |
| 1 | `/setup-profile` | Asks for your name, target job title, and a few preferences, then saves a profile. Run this first. |
| 2 | `/scan [count]` | Lists jobs with short, fluff-free summaries so you can quickly decide what to apply to. Default `count` is 10. |
| 3 | `/prepare-apply <job#>` | Produces a CV tailored to the selected job. |
| 4 | `/cover-letter <job#>` | Generates a cover letter for the selected job. |

## How it's built

- **Skills** (`.claude/skills/`) are the building blocks — each holds the logic for one step.
- **Commands** (`.claude/commands/`) are thin entry points that invoke the matching skill.
- **Mock data** lives in `data/mock-jobs.json`. A real deployment would replace the mock read with
  live job-source APIs / an MCP server (see the placeholder in `.mcp.json`, e.g.
  `/api/linkedinjobs?filter=…`, `/api/otherServiceJobs?filter=…`).

## Layout

```
.claude/
  commands/   # /setup-profile, /scan, /prepare-apply, /cover-letter
  skills/     # setup-profile, scan-jobs, prepare-apply, cover-letter
data/         # mock-jobs.json
profiles/     # your saved profile (gitignored)
output/       # tailored CVs & cover letters (gitignored)
```

## Try it

Open this folder in Claude Code and run:

```
/setup-profile
/scan 3
/prepare-apply 1
/cover-letter 1
```
