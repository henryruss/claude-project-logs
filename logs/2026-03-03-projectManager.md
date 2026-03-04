# projectManager

**Date:** 2026-03-03
**Location:** /Users/henryrussell/Projects/projectManager
**Stage:** in-progress
**GitHub:** https://github.com/henryruss/vibepm

## Summary
Unified the `/log` slash command with VibePM so that running `/log` at the end of any session automatically commits project code, detects the project lifecycle stage, writes a structured log with stage and GitHub URL metadata, updates all session context files (MEMORY.md, planning.md, CLAUDE.md), and pushes the log to trigger VibePM's webhook for automatic card creation/updates.

## Tech Stack
- Next.js 15 (App Router) + React 19 + TypeScript
- Tailwind CSS v4
- Supabase (Postgres + Auth + RLS)
- GitHub OAuth via Supabase Auth
- @dnd-kit/core + @dnd-kit/sortable
- Vercel (deployment)
- GitHub Webhooks (auto-sync)

## Files Built
- `~/.claude/commands/log.md` — rewrote `/log` command: now commits code, creates repos, detects stage, updates context files, writes structured logs
- `src/lib/parseProjectLog.ts` — added `stage` and `github_repo_url` fields to parser interface and extraction logic
- `src/app/api/webhook/github/route.ts` — webhook uses parsed stage (fallback "complete") and includes github_repo_url in insert/update
- `scripts/import-logs.ts` — import script uses parsed stage and github_repo_url
- `CLAUDE.md` — updated deployment status and parser description

## Key Decisions
- `/log` is the single end-of-session command — handles code commits, context updates, and log generation in one flow
- Webhook only overwrites stage if the parsed value is non-null, preventing accidental downgrades
- Backward compatible: old logs without Stage/GitHub fields parse as null and fall back to existing defaults

## Lessons Learned
- The `/log` command is a Claude Code slash command (stored in `~/.claude/commands/`), not a VibePM feature — it orchestrates across both the project repo and the logs repo
- Keeping parser changes backward-compatible (null defaults) avoids breaking existing imported logs
