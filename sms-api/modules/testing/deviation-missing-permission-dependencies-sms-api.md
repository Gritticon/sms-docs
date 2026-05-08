---
title: "Deviation: No explicit permission dependency on route handlers — sms-api"
type: deviation
project: sms-api
module: Permissions
last-updated: 2026-04-08
---


> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-03-21 (re-verified 2026-03-23).

## Deviation details

- **File:** `sms-api/app/api/routes/*.py` (scan across all route modules)
- **Violation type:** Write endpoints (`POST`, `PUT`, `PATCH`, `DELETE`) do not use an explicit `Depends(require_permission(...))` (or equivalent) pattern; `rg` found **no** matches for `require_permission` under `app/api/routes/`.
- **Expected pattern:** `docs/architecture/permission-implementation.md` — explicit write permissions via permission IDs; role embeds permissions; write actions should be enforceable server-side.
- **Actual:** `staff_bearer` / `get_staff_auth_context` (and similar) appear on many routes; fine-grained permission checks per route handler are not consistently visible in route code.

## Required action

- [ ] Confirm whether enforcement is delegated entirely to middleware or service layer; if so, document that in the architecture guide.
- [ ] If not, add explicit permission dependencies per write endpoint per `permission-implementation.md` and `permissions-adding-modules-checklist.md`.
- [ ] Confirm whether this is intentional (tech debt accepted) or must be fixed before release.

## Reference

- Architecture doc: `docs/architecture/permission-implementation.md`
