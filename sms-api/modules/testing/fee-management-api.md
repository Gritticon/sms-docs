---
title: "Fee Management API"
type: planning-stub
project: sms-api
module: Fee Management
last-updated: 2026-03-23
---

> Auto-promoted to testing on 2026-03-23 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — The largest API gap in the platform. No fee endpoints exist in the current SMS-API. Both SMS-UI (staff) and KYC (parent online payment) depend on this.

## What this API needs to cover

### Fee Structure
- `POST /fees/structures` — Create a fee structure (name, school, academic year, fee heads)
- `GET /fees/structures` — List fee structures for a school
- `PUT /fees/structures/{id}` — Update structure
- `DELETE /fees/structures/{id}` — Delete (only if no assignments made)

### Fee Heads (line items within a structure)
- `POST /fees/structures/{id}/heads` — Add fee head (e.g., Tuition, Transport, Activity)
- `PUT /fees/heads/{id}` — Update fee head (amount, due date, frequency)
- `DELETE /fees/heads/{id}` — Remove fee head

### Student Fee Assignment
- `POST /fees/assignments/bulk` — Assign a fee structure to a class/section (bulk)
- `POST /fees/assignments/{studentId}` — Assign to individual student (override)
- `GET /fees/assignments/{studentId}` — Get a student's current fee schedule
- `GET /fees/dues/{studentId}` — Get outstanding dues for a student (computed)

### Payments
- `POST /fees/payments` — Record a payment (studentId, amount, feeHead, paymentMethod, date, reference)
- `GET /fees/payments/{studentId}` — Payment history for a student
- `GET /fees/receipts/{paymentId}` — Receipt data (for PDF generation)

### Online Payment (KYC-triggered)
- `POST /fees/payments/online/initiate` — Start online payment session (returns payment gateway URL/token)
- `POST /fees/payments/online/callback` — Payment gateway webhook (mark payment confirmed)
- `GET /fees/payments/online/status/{sessionId}` — Poll payment status

### Reports
- `GET /fees/collection/summary` — Daily/monthly collection totals for a school
- `GET /fees/overdue` — Students with overdue amounts (filter by class/section/days overdue)

## Key decisions to make before writing spec

- Which payment gateway? (Razorpay / PayU / other — currently marked TBD)
- Partial payments allowed? (affects dues calculation logic)
- Concessions / scholarships — in scope for v1 or later?
- How are fee receipts numbered? (school-wide sequence? per academic year?)
- Multi-year fee history — same student across academic years?

## Related docs

- SMS-UI stub: `sms-ui/modules/needs-spec/fee-management`
- KYC spec: `/doc/kyc/specs/built/fees-payments-spec`
- KYC fees UX: `/doc/kyc/specs/testing/fees-ui-ux-design`
- API overview: `/doc/api-overview`
- Permission rules: `/doc/architecture/permission-implementation`
