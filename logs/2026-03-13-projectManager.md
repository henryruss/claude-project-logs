# VibePM (Project Manager Dashboard)

**Date:** 2026-03-13
**Location:** /Users/henryrussell/henmax/Projects/projectManager
**Stage:** in-progress
**GitHub:** https://github.com/henryruss/vibepm

## Summary
Implemented a full public portfolio mode so VibePM can be shared with recruiters and non-technical contacts without requiring login. Guests see a clean, read-only portfolio view under Henry's name; the owner logs in to get all edit controls back. Also drafted polished project summaries for all 6 projects to be pasted in via the detail panel.

## Tech Stack
- Next.js 15 (App Router) + React 19 + TypeScript
- Tailwind CSS v4
- Supabase (Postgres + RLS + Auth)
- GitHub OAuth via Supabase Auth
- Vercel deployment

## Files Built
- `src/app/page.tsx` — removed login redirect; passes `isOwner={!!user}` to Dashboard
- `src/hooks/useProjects.ts` — added `NEXT_PUBLIC_PORTFOLIO_USER_ID` fallback for anon project fetch
- `src/components/Dashboard.tsx` — accepts `isOwner`; rebrands header, hides admin chrome in public mode
- `src/components/portfolio/PortfolioGrid.tsx` — hides Add button, skips empty categories for guests
- `src/components/projects/ProjectDetailView.tsx` — full read-only mode: inputs → static text, todos/notes/delete hidden, links as anchors
- `supabase/public-read-policy.sql` — tracks manual SQL migration for public SELECT on owner's projects

## Key Decisions
- Single `isOwner={!!user}` prop threaded from page.tsx through component tree — avoids duplicating components
- Public fetch falls back to `NEXT_PUBLIC_PORTFOLIO_USER_ID` env var; all mutation functions already guarded with `if (!user) return`
- Supabase RLS policy opens SELECT on owner's projects to anon; todos/notes stay private
- Pipeline (kanban) view is internal-only — forced to portfolio view in public mode
- Empty category sections hidden in public mode (no "No agents yet" visible to recruiters)
- Subtle 🔒 login link replaces sign-out button for guests — owner can re-enter without it being obvious

## Lessons Learned
- Passing `undefined` to hooks that guard with `if (!user)` is a clean way to no-op them without changing their signatures
- `NEXT_PUBLIC_` prefix is required for env vars to be accessible in browser-side code in Next.js
- RLS policy needs to be run manually in Supabase SQL editor — can't be applied from code; tracking it in a `.sql` file in the repo is the right move for reference
