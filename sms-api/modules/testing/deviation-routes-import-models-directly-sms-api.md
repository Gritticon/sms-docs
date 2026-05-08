---
title: "Deviation: Route modules import models directly — sms-api"
type: deviation
project: sms-api
module: Layering
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-03-21.

## Deviation details

- **Files:** `sms-api/app/api/routes/students.py`, `school.py`, `staff.py`, `support_tickets.py`, `internal_employees.py`
- **Violation type:** Direct imports from `app.models` (or model submodules) inside route handlers.
- **Expected pattern:** `docs/architecture/project-rules.md` — business logic and data access in services; routes thin.
- **Actual:** Routes reference ORM models directly (e.g. for queries or typing) instead of only schemas + services.

## Required action

- [ ] Refactor each occurrence to go through `app/services/` where feasible.
- [ ] Document any intentional exceptions (e.g. trivial existence checks).

## Reference

- Architecture doc: `docs/architecture/project-rules.md`
