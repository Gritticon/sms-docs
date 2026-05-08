---
title: "Deviation: Missing permission check in attendance_buckets — sms-api"
type: deviation
project: sms-api
module: attendance_buckets
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-04-08.

## Deviation details

- **File:** `sms-api/app/api/routes/attendance_buckets.py`
- **Violation type:** Missing permission check
- **Expected pattern:** `permission` on write endpoints.
- **Actual:** Write endpoints detected with no permission dependency pattern.

## Required action

- [ ] Describe the specific fix needed
- [ ] Reference the architecture doc that defines the correct pattern
- [ ] Confirm whether this is intentional (tech debt accepted) or must be fixed before release

## Reference

- Architecture doc: `docs/architecture/permission-implementation.md`
