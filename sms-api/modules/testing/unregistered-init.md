---
title: "Deviation: Unregistered route __init__ — sms-api"
type: deviation
project: sms-api
module: __init__
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-04-08.

## Deviation details

- **File:** `sms-api/app/api/routes/__init__.py`
- **Violation type:** Unregistered route
- **Expected pattern:** Route modules should be included in application router registration.
- **Actual:** Route file exists but no registration detected in `app/main.py` or `app/api/__init__.py`.

## Required action

- [ ] Describe the specific fix needed
- [ ] Reference the architecture doc that defines the correct pattern
- [ ] Confirm whether this is intentional (tech debt accepted) or must be fixed before release

## Reference

- Architecture doc: `docs/architecture/system-architecture.md`
