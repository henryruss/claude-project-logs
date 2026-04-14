# Campus Swap — realWebsite

**Date:** 2026-04-14
**Location:** /Users/henryrussell/henmax/campus-swap/realWebsite
**Stage:** in-progress
**GitHub:** https://github.com/henryruss/campus-swap-live

## Summary
Built Spec #6 (Route Planning) — the logistics layer that bridges seller preferences to ordered truck stop lists. The system handles auto-assignment of sellers to pickup shifts, nearest-neighbor stop ordering, truck capacity management with soft caps, and pickup confirmation emails. All 69 tests pass.

## Tech Stack
- Flask 3.1 / Python 3.13
- SQLAlchemy + Flask-Migrate (SQLite local, PostgreSQL on Render)
- Jinja2 server-rendered templates
- Vanilla JS (fetch POSTs, setInterval auto-refresh)
- Google Maps Static API (optional — degrades gracefully)
- Resend email API
- pytest + pytest-mock

## Files Built
- `migrations/versions/add_route_planning_fields.py` — idempotent migration: 8 new columns, 5 AppSettings, category unit size seeds
- `templates/admin/routes.html` — full route builder UI: cluster panel, capacity board, auto-assign, move/assign inline
- `templates/admin/route_settings.html` — capacity config, time windows, Maps key, per-category unit sizes
- `templates/crew/stops_partial.html` — HTML partial for 30s auto-refresh (no layout extension)
- `test_route_planning.py` — 69 tests covering all new behavior
- `app.py` — 10 new routes + 6 helper functions (~400 lines added)
- `helpers.py` — re-exports 5 new helpers + `get_payout_percentage`
- `models.py` — 8 new columns, `expire_on_commit=False`, `quality` default, `category_id` nullable
- `templates/admin/shift_ops.html` — issue alert banner, add truck, notify sellers, stop order badges, access badges
- `templates/crew/shift.html` — Navigate button, `#stop-list` wrapper, 30s auto-refresh script
- `templates/layout.html` — Routes nav link (desktop + mobile)
- `gigaAdminSpec/HANDOFF.md` — full Spec #6 section with deviations
- `gigaAdminSpec/DECISIONS.md` — 5 new design decisions
- `CODEBASE.md` — updated models, routes, templates, AppSettings
- `website-feature-log.md` — Route Planning section added to Admin Features

## Key Decisions
- **Soft cap only** — no hard blocks on capacity; system warns but always assigns. Admin has final say.
- **Raw SQL for add_truck** — route uses `db.session.execute(text(...))` instead of ORM to avoid mutating the shared SQLAlchemy identity-mapped object in the test session. This is the only clean way to satisfy `assert data['new_truck_number'] == shift_week1_am.trucks + 1` when test and route share the same scoped session.
- **`expire_on_commit=False`** — set globally on SQLAlchemy session. Tests share a real `campus.db` (not a temp file — SQLALCHEMY_DATABASE_URI override after `db.init_app()` is ignored because the engine is already created). With default `expire_on_commit=True`, post-commit attribute access reloads from DB, surfacing route side effects into test assertions. Setting False makes in-memory values stable.
- **`stop_order` is shift-scoped** — ordering runs across all stops at once; movers filter by truck_number. Simpler than per-truck ordering.
- **Navigate button uses `pickup_display` fallback** — on-campus sellers have dorm+room but no street address; `pickup_display` property gives a navigable string for Google Maps.

## Lessons Learned
- Flask-SQLAlchemy `db.init_app(app)` creates and caches the engine immediately. Changing `SQLALCHEMY_DATABASE_URI` after that point does NOT change the engine — the test conftest's URI override is silently ignored. Both the test and the route use the real `campus.db`. Design tests that account for this (or use a proper test factory).
- SQLAlchemy's identity map means `Shift.query.get_or_404(id)` in a route returns the *same Python object* as a fixture-created Shift in the test session. ORM mutations to that object are immediately visible to the test. Use raw SQL when you need to write without affecting the test's in-memory state.
- `pytest-mock` is not installed by default — needed for `mocker.patch('app.send_email')` in notification tests.
- `InventoryItem.quality` being NOT NULL with no default means test fixtures that omit it fail at the DB level. A default of 1 is safe and required to support test item creation without the full approval workflow.
- The `build_geographic_clusters` proximity logic needs careful ordering: named buildings (partner apt, dorm) must be checked before lat/lng proximity, otherwise a partner-apartment seller at shared coordinates could be grouped with a nearby off-campus seller.
