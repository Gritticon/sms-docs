---
title: "Complaints Module — SMS-UI"
type: planning-stub
project: sms-ui
module: Complaints
last-updated: null
---

## Status

**Needs spec** — Feature is in the product roadmap (Module 8) but has no SMS-UI implementation specification.

## What this feature is

The Complaints module allows school staff to receive, manage, and resolve complaints submitted by parents through the KYC app:
- View incoming complaints (all or filtered by status/category)
- Assign complaints to staff members
- Update status (Open → In Progress → Resolved)
- Add internal notes or reply to parent
- View resolution history

The parent-side complaint submission and status tracking is already specced in KYC — this stub covers the **staff resolution UI**.

## What needs to be planned

- [ ] Complaints inbox screen (list with filters: status, category, date, student)
- [ ] Complaint detail view (full thread: parent message + staff responses)
- [ ] Status workflow (Open → In Progress → Resolved → Closed)
- [ ] Assignment of complaint to staff member
- [ ] Internal notes (visible to staff only, not parent)
- [ ] Reply to parent (visible in KYC app)
- [ ] Category management (who defines complaint categories?)
- [ ] Notifications to staff when a new complaint arrives
- [ ] Role-based access (class teacher sees only their students' complaints?)
- [ ] SLA / escalation rules (optional for MVP?)

## Related docs

- KYC side spec: `/doc/kyc/specs/built/complaints-spec`
- Module registry: `/doc/sms-ui/architecture/modules-and-submodules`
- Feature matrix: `/doc/product/feature-matrix`
- API needed: `sms-api/modules/testing/complaints-api`

## Dependencies

- SMS-API Complaints endpoints (also needs-spec)
- Communication Hub (staff reply goes through messaging infrastructure)
- RBAC permission IDs for Complaints module
