---
title: "Engagement views — kyc"
type: planning-stub
project: kyc
module: Engagement
last-updated: 2026-03-28
---

> Auto-promoted to testing on 2026-03-28 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**In testing** — Spec coverage still thin; implementation in `kyc/lib/views/engagement/`. Promoted from `needs-spec` on 2026-03-28 after pipeline sync.

## What was found in code

- **File(s):** `kyc/lib/views/engagement/engagement_views.dart`
- **Type:** Flutter view file(s) — parent/student app engagement feature area
- **Detected details:** Directory `kyc/lib/views/engagement/` contains at least one view file; exact screens and widgets not yet inspected in depth.

## What needs to be documented

- [ ] What engagement features are included (push notifications, polls, event feeds, announcements, etc.)
- [ ] Which user roles see engagement content (parent vs student)
- [ ] API endpoints / repositories used
- [ ] Navigation entry points (how users reach engagement from home or nav)
- [ ] Whether this overlaps with `communication-spec.md` or `notifications-events-spec.md` in docs/kyc/specs/built/

## Open questions

- Is this a distinct feature area or a sub-view of the Communications module?
- Is the engagement feature GA or still experimental?
- Should this be documented as a separate spec or merged into the notifications/communication spec?
