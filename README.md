# AgenticJob

Simplify the job search and application process with automation and agentic workflows, driven
through **Claude Code** slash commands and skills.

> Job postings come from the `job-sources` MCP server and CVs are rendered by the `cv-renderer` MCP
> server — no mock data, no fabricated content. See `CLAUDE.md` for the full scope.

## The workflow

| Step | Command | What it does |
| ---- | ------- | ------------ |
| 1 | `/setup-profile` | Collects your profile plus a required CV upload; saves `profiles/profile.json` and a baseline `profiles/cv/cv.json`. Run this first. |
| 2 | `/scan [count]` | Fetches postings from a `job-sources` provider, scores them against your profile, and lists the best `count` (default 10). |
| 3 | `/prepare-apply <job# \| job-url>` | Builds `output/<company>-<role>/` with a tailored CV (JSON + PDF, approval-gated before rendering) and, on request, a cover letter. |
| 4 | `/cover-letter <job# \| job-url>` | Writes/refreshes just the cover letter for that job. |

## How it's built

- **Skills** (`.claude/skills/`) hold the logic: `setup-profile`, `scan-jobs`, `score-jobs`,
  `prepare-apply`, `tailor-cv`, `cover-letter`.
- **Commands** (`.claude/commands/`) are thin entry points that invoke the matching skill.
- **MCP servers** (`.mcp.json`) supply the data: `job-sources` (search/fetch postings) and
  `cv-renderer` (renders CV JSON to PDF).

## Dependencies

Both MCP servers are separate projects you need running somewhere reachable:

- [job-sources — JobsProviderMcp](https://github.com/ivan213k/JobsProviderMcp)
- [cv-renderer — JsonToCvMCP](https://github.com/ivan213k/JsonToCvMCP)

Deploy them (locally or otherwise) and set their URLs in `.mcp.json`. Without them, `/scan` and the
CV-rendering step of `/prepare-apply` will report the server as unreachable rather than falling back
to mock data.

## Layout

```
.claude/
  commands/   # /setup-profile, /scan, /prepare-apply, /cover-letter
  skills/     # setup-profile, scan-jobs, score-jobs, prepare-apply, tailor-cv, cover-letter
profiles/     # your profile + baseline CV (gitignored)
output/       # per-job application packages: job.md, cv.json, CV.pdf, cover-letter.txt (gitignored)
```

## Try it

Open this folder in Claude Code and run:

```
/setup-profile
/scan 3
/prepare-apply 1
/cover-letter 1
```
