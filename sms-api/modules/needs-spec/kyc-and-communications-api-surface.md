---
title: "KYC and communications API surface — sms-api"
type: planning-stub
project: sms-api
module: kyc parent app + staff comms
last-updated: null
---

## Status

**Needs spec** — Implementation detected in codebase during pipeline sync on 2026-04-25, but no `docs/sms-api/` specification exists for these route modules (zero filename matches under `docs/sms-api/` at time of scan).

## What was found in code

- **Type:** FastAPI `APIRouter` modules under `sms-api/app/api/routes/`, all registered in `sms-api/app/main.py`.
- **Files (KYC / parent app):** `kyc_auth.py` (prefix `/kyc/auth`), `kyc_school.py`, `kyc_student.py`, `kyc_attendance.py`, `kyc_marks.py`, `kyc_timetable.py`, `kyc_fees.py` (e.g. `/kyc/fees`), `kyc_diary.py`, `kyc_leaves.py`, `kyc_complaints.py`.
- **Files (KYC + staff comms bridge):** `kyc_announcements.py`, `kyc_messages.py` (parent-facing; tags typically `kyc-*`).
- **Files (staff comms, shared tables):** `communications_announcements.py`, `communications_messages.py` (e.g. prefix `/communications/messages`).

## What needs to be documented

- [ ] Per-domain or grouped API contract (request/response, pagination, idempotency)
- [ ] Auth model: parent JWT vs staff bearer per route group; school scoping
- [ ] Permission and multi-tenant safety for staff write paths
- [ ] Mapping from these routes to KYC Flutter modules and to `docs/kyc/specs/` product specs
- [ ] How `communications_*` relates to `kyc_*` announcement/message routes (single source of truth for message rows)

## Open questions

- Should KYC API docs live only under `docs/sms-api/` or cross-link from `docs/kyc/specs/testing/` as the human-readable layer?
- Which routes are stable for external integrators vs internal app-only?
