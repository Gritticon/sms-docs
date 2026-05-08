---
title: "Early Permission Config — sms-api"
type: planning-stub
project: sms-api
module: early permission config
last-updated: 2026-04-25
---

> Auto-promoted to testing on 2026-04-25 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Implementation detected in codebase during pipeline sync on 2026-04-12, but no specification exists.

## What was found in code

- **File(s):** `sms-api/app/api/routes/early_permission_config.py`
- **Type:** Route file (FastAPI APIRouter)
- **Detected endpoints/screens:** Early permission configuration endpoints (exact HTTP methods and paths to be confirmed from route file; likely CRUD operations for configuring early-departure / early-permission rules per school)

## What needs to be documented

- [ ] Full feature specification (what does this do, who uses it, why)
- [ ] API contract (request/response shapes, error codes)
- [ ] Permission model (who can access this)
- [ ] Edge cases and validation rules
- [ ] Integration with other modules (related to early_permission_requests.py and leave management)

## Open questions

- Is this feature complete and intentional, or work-in-progress?
- Does it follow the documented architecture patterns (permission enforcement on write endpoints)?
- Should it be merged into a broader leave/attendance configuration doc or kept as a standalone spec?
