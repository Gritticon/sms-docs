---
title: "Complaints API"
type: planning-stub
project: sms-api
module: Complaints
last-updated: 2026-03-23
---

> Auto-promoted to testing on 2026-03-23 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — No complaints endpoints exist in the current SMS-API. SMS-UI (staff resolves) and KYC (parent submits and tracks) both depend on this.

## What this API needs to cover

### Complaint Submission (KYC-side)
- `POST /complaints` — Submit complaint (studentId, category, subject, body, attachments?)
- `GET /complaints/categories` — List complaint categories for a school

### Staff Resolution (SMS-UI-side)
- `GET /complaints` — List complaints (filter by status, category, school, class, date)
- `GET /complaints/{id}` — Get complaint detail with full thread
- `PUT /complaints/{id}/status` — Update status (OPEN → IN_PROGRESS → RESOLVED → CLOSED)
- `PUT /complaints/{id}/assign` — Assign to staff member
- `POST /complaints/{id}/notes` — Add internal note (visible to staff only)
- `POST /complaints/{id}/reply` — Reply to parent (visible in KYC)

### Categories (Admin)
- `POST /complaints/categories` — Create category (name, school)
- `PUT /complaints/categories/{id}` — Update
- `DELETE /complaints/categories/{id}` — Delete

## Key decisions to make before writing spec

- Does a reply to a complaint go through the Communication Hub (messages), or is it a separate flow?
- Attachments in v1? (photos from parents as evidence)
- Can a parent submit multiple complaints for the same issue?
- Notification to staff on new complaint — push notification or just in-app badge?

## Related docs

- SMS-UI stub: `sms-ui/modules/needs-spec/complaints`
- KYC spec: `/doc/kyc/specs/built/complaints-spec`
- API overview: `/doc/api-overview`
