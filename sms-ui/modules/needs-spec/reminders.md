---
title: "Reminders — SMS-UI"
type: planning-stub
project: sms-ui
module: Reminders
last-updated: null
---

## Status

**Needs spec** — Feature is referenced in the product roadmap and the KYC spec exists for the parent-facing view, but there is no SMS-UI spec for how staff configure or trigger reminders.

## What this feature is

The Reminders module allows school admins to configure and send automated or manual reminders to parents via the KYC app:
- Fee payment reminders (auto-triggered on due date or overdue)
- Event reminders (upcoming school events, exam dates)
- Transport fee reminders
- Custom one-off reminders from admin

The parent-side reminder inbox is already specced in KYC — this stub covers the **staff-side reminder management UI**.

## What needs to be planned

- [ ] Reminder types (Fee, Event, Transport, Custom — extensible?)
- [ ] Automated reminder rules (e.g., send 3 days before fee due date, then on due date, then 1 week overdue)
- [ ] Manual reminder screen (select template → select audience → send)
- [ ] Audience targeting (whole school / by class / by section / by student)
- [ ] Reminder history log (what was sent, when, to how many)
- [ ] Template management (pre-defined reminder text, editable)
- [ ] Channel (push notification only? Email? SMS?)
- [ ] Permission model (who can send reminders)
- [ ] Integration with Fee Management (auto-trigger when overdue threshold hit)

## Related docs

- KYC side spec: `/doc/kyc/specs/built/reminders-spec`
- Feature matrix: `/doc/product/feature-matrix`
- API needed: `sms-api/modules/needs-spec/reminders-api`
- Communication Hub (reminders may use the same delivery infrastructure)

## Dependencies

- SMS-API Reminders endpoints (also needs-spec)
- Fee Management (for automated fee reminders — needs fee management to be built first or in parallel)
- Communication Hub infrastructure (delivery channel)
