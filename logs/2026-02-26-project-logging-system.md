# Project Logging System

**Date:** 2026-02-26
**Location:** ~/.claude/commands/

## Summary
Built a lightweight system for automatically logging every Claude Code project to a GitHub repo. Instead of manually copying summaries into a Word doc, a single `/log` command generates a structured markdown entry and pushes it to GitHub. A companion `/projects` command reads all past logs and returns a bullet-point history of everything built.

## Tech Stack
- Claude Code custom slash commands (markdown prompt files)
- Git + GitHub (storage and version control)
- Bash (git add/commit/push in the command)

## Files Built
- `~/.claude/commands/log.md` — `/log` command: reads project context, writes structured log entry, commits and pushes to GitHub
- `~/.claude/commands/projects.md` — `/projects` command: reads all log files and returns bullet-point summaries
- `~/Projects/claude-project-logs/logs/` — GitHub repo folder where all project logs are stored

## Key Decisions
- **Global commands in `~/.claude/commands/`** — available in every project, not just one
- **One file per project** named `YYYY-MM-DD-{project-name}.md` so logs sort chronologically by filename
- **Reads auto-memory MEMORY.md** to source project details automatically, so `/log` works without manual input

## Lessons Learned
- Claude Code custom commands live in `~/.claude/commands/` as plain `.md` files — the file content becomes the prompt Claude receives when the command is invoked
- `$ARGUMENTS` substitutes anything typed after the command name
- The auto-memory path is deterministic: `~/.claude/projects/{cwd-with-slashes-as-dashes}/memory/MEMORY.md`
