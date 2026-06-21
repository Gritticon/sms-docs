---
title: "Communication Hub — SMS-UI"
type: planning-stub
project: sms-ui
module: Communication Hub
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Gap analysis complete** — Full multi-layer gap analysis produced 2026-05-29.
See [`communication-hub-gap-analysis.md`](communication-hub-gap-analysis.md) for all findings,
priority table, and recommended sprint sequence (Sprints 1–6).

**Implementation detected, QA not signed off.** 35+ gaps identified across API, UI, design,
dashboard integration, cross-project linkage, and security. 4 P0 bugs require fix before QA.

## What this feature is

The Communication Hub allows school staff to reach parents and students directly from the SMS staff portal:
- Compose and send messages to individual parents or groups
- Create announcements broadcast to a whole school, class, or section
- Write posts that appear in the KYC home feed
- View message threads and reply

The parent/student side is already specced in KYC — this stub covers the **staff-side UI** only.

## What needs to be planned

- [ ] Navigation entry point (where does it sit in the SMS-UI module list?)
- [ ] Message composition screen (recipients: by class, section, student, or individual)
- [ ] Announcements screen (create, schedule, audience targeting)
- [ ] Post creation (feed post for KYC home page)
- [ ] Inbox/sent view for staff (threaded view?)
- [ ] Permission rules — who can send school-wide vs. class-only vs. individual
- [ ] Attachment support (yes/no for MVP?)
- [ ] Read receipts / delivery status visibility for staff
- [ ] Integration with the existing 180-permission RBAC matrix (which permission IDs apply?)

## Related docs

- KYC side spec: `/doc/kyc/specs/built/communication-spec`
- KYC notifications: `/doc/kyc/specs/built/notifications-events-spec`
- Module & permission registry: `/doc/sms-ui/architecture/modules-and-submodules`
- Feature matrix: `/doc/product/feature-matrix`
- API needed: `sms-api/modules/needs-spec/communication-hub-api`

## Dependencies

- SMS-API Communication Hub endpoints (also needs-spec — must be planned in parallel)
- RBAC permission IDs for Communication module to be allocated in the permission registry
