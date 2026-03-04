# VibePM — Portfolio Restructure

**Date:** 2026-03-03
**Location:** /Users/henryrussell/Projects/projectManager
**Stage:** in-progress
**GitHub:** https://github.com/henryruss/vibepm
**Category:** website

## Summary
Transformed VibePM from a kanban task board into an AI project portfolio headquarters. Added two-category organization (Websites/Webapps and Agents) with a PortfolioGrid as the primary view, per-project todos and notes, an AddProjectModal, and PDF export for both individual projects and full portfolio summaries.

## Tech Stack
- Next.js 15 (App Router) + React 19 + TypeScript
- Tailwind CSS v4
- @dnd-kit/core + @dnd-kit/sortable
- jspdf (new)
- Supabase (Postgres + Auth + RLS)
- Vercel

## Files Built
- `src/components/portfolio/PortfolioGrid.tsx` — Main portfolio view with category sections, project counts, deployed counts
- `src/components/portfolio/PortfolioCard.tsx` — Rich project card with status badge, tech stack pills, GitHub/URL links, relative timestamps
- `src/components/portfolio/AddProjectModal.tsx` — New project modal with category and stage selectors
- `src/lib/exportPdf.ts` — PDF generation for single project reports and full portfolio summaries
- `src/lib/projectUtils.ts` — Status helpers: isDeployed, getStatusLabel, getStatusColor, getRelativeTime
- `src/lib/types.ts` — Added ProjectCategory type, category field, project_id on TodoItem/NoteData
- `src/components/Dashboard.tsx` — Replaced sidebar layout with Portfolio/Pipeline view toggle + export button
- `src/components/projects/ProjectDetailView.tsx` — Expanded to max-w-2xl with category/stage editing, GitHub URL, per-project todos/notes, export PDF
- `src/hooks/useProjects.ts` — Category support in addProject + getCategoryProjects filter
- `src/hooks/useSupabaseTodos.ts` — Optional projectId filtering for per-project scoping
- `src/hooks/useSupabaseNotes.ts` — Optional projectId filtering for per-project scoping
- `src/lib/parseProjectLog.ts` — Parse **Category:** field from log markdown
- `src/app/api/webhook/github/route.ts` — Pass category through on upsert
- `src/app/globals.css` — Category colors (teal/purple), view transition animation, print styles
- `supabase/schema.sql` — Added category column, project_id FK on todos/notes

## Key Decisions
- Portfolio grid is the default view; kanban preserved as togglable "Pipeline" secondary view
- Todos and notes moved from global sidebar to per-project (project_id FK with cascade delete)
- Category defaults to "website" for backward compatibility with existing projects
- PDF export uses dynamic import to lazy-load jspdf only when needed
- Webhook only sets category if parsed value is valid ("website" or "agent")

## Lessons Learned
- Tailwind v4 does not resolve dynamic class names like `border-${variable}` — must use pre-defined style maps with full class strings
- JSX.Element type is not available in all TypeScript configs — use React.ReactNode instead
- Per-project notes need the same "create if not exists" pattern as the global note (PGRST116 error handling)
