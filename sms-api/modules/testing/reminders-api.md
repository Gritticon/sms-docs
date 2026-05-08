---
title: "Reminders API"
type: planning-stub
project: sms-api
module: Reminders
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — No reminders endpoints exist in the current SMS-API. The KYC parent-side reminder display is already built; this is the API that creates and schedules them.

## What this API needs to cover

### Reminder Templates
- `POST /reminders/templates` — Create template (type, title, body with variables like {{studentName}}, {{dueDate}})
- `GET /reminders/templates` — List templates for a school
- `PUT /reminders/templates/{id}` — Update template
- `DELETE /reminders/templates/{id}` — Delete template

### Reminder Rules (automated)
- `POST /reminders/rules` — Create an automated rule (e.g., trigger: FEE_OVERDUE, offset: +3 days, template, audience)
- `GET /reminders/rules` — List rules for a school
- `PUT /reminders/rules/{id}` — Update rule
- `DELETE /reminders/rules/{id}` — Delete rule (disable automation)

### Manual Reminders (one-off)
- `POST /reminders/send` — Send a reminder now (templateId, audience: {school/class/section/students[]})
- `GET /reminders/history` — Log of all sent reminders (filter by type, date, audience)

### Parent-facing (KYC reads)
- `GET /reminders/student/{studentId}` — Reminders for a student/parent (paginated, most recent first)
- `PUT /reminders/{id}/read` — Mark reminder as read

## Automated trigger types to plan
- `FEE_DUE` — N days before fee due date
- `FEE_OVERDUE` — N days after fee due date passes unpaid
- `EXAM_UPCOMING` — N days before exam date
- `EVENT` — N days before a calendar event
- `TRANSPORT_FEE_DUE` — Same as FEE_DUE but transport-specific
- `CUSTOM` — Manual one-off

## Key decisions to make before writing spec

- Delivery channel for v1: push notification only, or also email/SMS?
- Who processes the scheduled jobs? (Cron in ECS, or separate worker?)
- Can a school customise template text, or is it fixed?

## Related docs

- SMS-UI stub: `sms-ui/modules/needs-spec/reminders`
- KYC spec: `/doc/kyc/specs/built/reminders-spec`
- Communication Hub API (may share push delivery): `sms-api/modules/needs-spec/communication-hub-api`
- Fee Management API (triggers fee reminders): `sms-api/modules/testing/fee-management-api`
