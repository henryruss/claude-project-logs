# VibePM — Alpine to Ocean Panorama + Webhook Auto-Sync

**Date:** 2026-03-03
**Location:** /Users/henryrussell/Projects/projectManager

## Summary
Redesigned VibePM's UI from the subtle "Alpine Morning" theme to a bold "Alpine to Ocean Panorama" — a vivid 4-biome SVG landscape (snow mountains → evergreen forest → beach → ocean) that dominates the bottom 45% of the kanban board, with glassmorphic columns and cards floating above it. Also added a GitHub webhook API route so pushing new project logs to the claude-project-logs repo automatically syncs them into VibePM via Supabase.

## Tech Stack
- Next.js 15 (App Router) + React 19 + TypeScript
- Tailwind CSS v4
- @dnd-kit/core + @dnd-kit/sortable (drag-and-drop)
- Supabase (Postgres + Auth + RLS)
- Vercel (deployment)
- GitHub Webhooks (HMAC-SHA256 signature verification)
- Inline SVG (hand-crafted panoramic landscape)
- CSS keyframe animations (wave drift, palm frond sway)
- Glassmorphism (backdrop-filter blur + rgba backgrounds)

## Files Built
- `src/components/landscape/PanoramaSVG.tsx` — New 1200x400 inline SVG with 4 biome zones: snow-capped mountains with ski poles/skis, evergreen pine forest, beach with palm trees and colorful surfboards, animated ocean waves with sea foam
- `src/app/globals.css` — Added wave-drift and palm-sway keyframe animations, .column-glass/.card-glass utility classes, responsive media query to hide panorama below 1024px
- `src/app/layout.tsx` — Removed all old decorative SVGs (landscape silhouette, surfboard, ski poles, mountain peak, wave curl), kept ambient overlays
- `src/components/kanban/KanbanBoard.tsx` — Replaced flex layout with relative container + absolute panorama layer (bottom 45vh) + CSS grid-cols-4 for columns, imported PanoramaSVG
- `src/components/kanban/KanbanColumn.tsx` — Glassmorphic column styling (rgba 0.65 + blur 8px), removed fixed widths for grid fill, thicker 3px color strip, scrollable card area (max-h 55vh)
- `src/components/kanban/KanbanCard.tsx` — Semi-transparent card background (rgba 0.8 + blur 4px), softer border opacity, drag overlay stays opaque
- `src/components/Dashboard.tsx` — Removed bg-void-soft from main area so landscape shows through
- `src/app/api/webhook/github/route.ts` — New POST handler for GitHub push webhooks: verifies HMAC-SHA256 signature, processes push events to main branch, fetches changed .md files from logs/ directory via raw GitHub URL, parses with parseProjectLog(), upserts into Supabase (dedupes on source_log_path)

## Key Decisions
- Panorama is absolutely positioned at bottom 45vh with z-0, columns sit in z-10 grid above — keeps landscape visible while cards don't cover it
- Used CSS grid (grid-cols-4) instead of flex for columns so they fill available space without fixed widths
- Glassmorphic approach: column-glass at 0.65 opacity + 8px blur, card-glass at 0.8 opacity + 4px blur — enough transparency to see landscape, enough opacity to read text
- Panorama hidden on screens below 1024px via media query (cards need the space)
- Webhook uses crypto.timingSafeEqual for signature verification to prevent timing attacks
- Webhook upserts (updates existing, inserts new) rather than skip-on-duplicate, so log edits propagate
- SVG uses preserveAspectRatio="xMidYMax meet" to anchor landscape at bottom regardless of container size

## Lessons Learned
- Inline SVG in React needs careful attention to camelCase attributes (strokeWidth, strokeLinecap, etc.) — easy to miss
- Glassmorphism with backdrop-filter can cause performance issues if overused; limiting it to columns and cards (not the whole page) keeps it smooth
- CSS grid-cols-4 works great with dnd-kit — the library uses DOM rects for collision detection so the layout method doesn't matter
- GitHub webhook signature uses `sha256=` prefix (not `sha1=`) for the newer x-hub-signature-256 header
- Next.js API routes need to read request.text() before parsing JSON when you need the raw body for signature verification
