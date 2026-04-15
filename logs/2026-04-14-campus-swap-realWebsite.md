# Campus Swap — realWebsite

**Date:** 2026-04-14
**Location:** /Users/henryrussell/henmax/campus-swap/realWebsite
**Stage:** in-progress
**GitHub:** https://github.com/henryruss/campus-swap-live

## Summary
Implemented Spec #8 — Seller Rescheduling: a token-gated and auth-gated flow that lets sellers pick a new pickup slot without admin intervention. Built on top of Spec #6 (Route Planning). Also fixed several bugs discovered during testing: tracker message ordering, address display using wrong field for on-campus sellers, naive datetime comparison with SQLite, and seller readiness gating.

## Tech Stack
- Python 3.13 / Flask 3.1
- SQLAlchemy + Flask-Migrate (PostgreSQL in prod, SQLite locally)
- Jinja2 server-rendered templates
- Vanilla JS (no React)
- Resend API (email)
- pytest (69/69 route planning tests passing)

## Files Built
- `migrations/versions/add_seller_rescheduling.py` — idempotent migration: reschedule_token table, new ShiftPickup/Shift columns, 3 AppSettings
- `templates/seller/reschedule.html` — full pickup-window week grid, Mon–Sun columns, prev/next week nav, radio cards
- `templates/seller/reschedule_confirm.html` — shared success/error confirmation page
- `app.py` — 8 new helpers, 4 new routes, 6 modified routes, 2358 insertions total
- `models.py` — RescheduleToken model, new ShiftPickup/Shift fields
- `test_route_planning.py` — fixture updates for has_pickup_location requirement
- `gigaAdminSpec/HANDOFF.md` — Spec #8 section added
- `gigaAdminSpec/DECISIONS.md` — 7 new decisions documented
- `gigaAdminSpec/SPEC_CHECKLIST.md` — all 99 Spec #8 checks marked ✅
- `CODEBASE.md` — RescheduleToken model, new routes, new templates, AppSettings

## Key Decisions
- Reschedule eligibility is window-based (PICKUP_WEEK_DATE_RANGES) not Shift-record-based — admin doesn't need to pre-create all shifts
- Shifts auto-created on demand when seller picks an unmapped date/slot
- reschedule_max_weeks_forward='0' means no cap (seller can pick any future date in window)
- Naive datetimes in RescheduleToken — SQLite reads datetimes without tzinfo, comparing to timezone-aware _now_eastern() raises TypeError
- has_pickup_location gate added to all assign paths — sellers without complete address can't be assigned
- Pickup Window stat cell locks as soon as ShiftPickup exists, not just after notification email is sent

## Lessons Learned
- `_compute_seller_tracker` messages were one stage off: each message described what happened *at* a milestone but was shown while *waiting for* it. Seller with ShiftPickup saw "Driver has your items" because active stage advanced to `picked_up`.
- `pickup_address` is only set for off_campus_other sellers — on_campus and off_campus_complex sellers use pickup_dorm/room. Ops page and mover view were showing "No address on file" for on-campus sellers. Fix: use `seller.pickup_display` property which handles all 3 location types.
- Test fixtures for route planning lacked pickup_access_type/pickup_floor/pickup_room after we added has_pickup_location gating — 8 tests broke, fixed by updating fixtures.
- SQLite stores datetimes as naive strings; Python's _now_eastern() returns timezone-aware. Any DB datetime comparison needs .replace(tzinfo=None).
