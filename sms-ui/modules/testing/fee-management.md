---
title: "Fee Management — SMS-UI"
type: planning-stub
project: sms-ui
module: Fee Management
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Feature is in the product roadmap but has no SMS-UI implementation specification. The KYC parent-side payment UI is already specced.

## What this feature is

Fee Management is the staff-side module for configuring, assigning, and tracking school fees:
- Define fee structures (tuition, transport, activity, exam fees)
- Assign fee structures to students (by class, section, or individually)
- Record payments (cash, cheque, online)
- View dues and overdue balances per student
- Generate receipts and download fee statements
- Track collection totals across the school

The parent-side (view dues, pay online) is already specced in KYC — this stub covers the **staff collection and management UI**.

## What needs to be planned

- [ ] Fee structure setup (create fee heads: tuition, transport, activity; set amounts)
- [ ] Fee term configuration (monthly/quarterly/annual; due dates)
- [ ] Student fee assignment (bulk assign by class or individual override)
- [ ] Payment recording screen (student search → select fee head → record payment → print/email receipt)
- [ ] Fee collection dashboard (total collected today, this month, overdue summary)
- [ ] Overdue tracker (list of students with outstanding dues, send reminders)
- [ ] Receipt generation (PDF, numbered, with school branding)
- [ ] Fee statements (student-wise full statement for the year)
- [ ] Concessions / discounts (scholarship, sibling discount — yes/no MVP?)
- [ ] Online payment tracking (payments made via KYC app, auto-recorded)
- [ ] Export (fee collection report to Excel/PDF)
- [ ] Permission model (who can record payments vs. view-only vs. configure structures)

## Related docs

- KYC side spec: `/doc/kyc/specs/built/fees-payments-spec`
- KYC fees UX improvements: `/doc/kyc/specs/testing/fees-ui-ux-improvement-brief`
- KYC fees design: `/doc/kyc/specs/testing/fees-ui-ux-design`
- Feature matrix: `/doc/product/feature-matrix`
- API needed: `sms-api/modules/testing/fee-management-api`

## Dependencies

- SMS-API Fee Management endpoints (also needs-spec — the largest API gap)
- Payment gateway integration (for online payment reconciliation)
- Student Management module (already built — student selection)
- School Management (fee settings: late fees, grace period, partial payment rules)
