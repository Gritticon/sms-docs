---
title: Fees and Payments Spec
type: spec
project: kyc
module: Fees
last-updated: 2026-03-18
---

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Fees & Invoices — Specification

## 1. Summary
The Fees & Invoices module enables parents to pay school fees online for each child, view invoices and payment history, and receive clear visibility into upcoming and overdue payments. All fee structures, due dates, discounts, and payment processing logic reside in the SMS backend; KYC presents a simple, trustworthy UI to initiate and track payments.

This module aims to reduce in-person fee payments and paper invoices by offering a convenient digital alternative aligned with modern payment options.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display a list of fee items or schedules for the active child (e.g., term fees, transport fees, other charges) with key details (name, amount, due date, status) from SMS API. | P0 |
| F2 | System shall clearly distinguish between paid, partially paid, overdue, and upcoming fees based on backend status. | P0 |
| F3 | System shall allow parents to initiate online payment for one or more outstanding fee items, delegating all payment processing to SMS API and its payment gateway integrations. | P0 |
| F4 | System shall display confirmation of payment status after a payment attempt (success, failure, pending) using information returned by SMS API. | P0 |
| F5 | System shall present an invoice or receipt view per payment, including amount, date, items covered, and transaction reference as provided by the backend. | P0 |
| F6 | System shall provide a payment history list for the active child, with filters (e.g., date range, status) where supported by the backend. | P1 |
| F7 | System shall support multi-child parents by allowing them to see and pay fees separately for each child based on the active context. | P0 |
| F8 | System shall trigger or surface fee reminders (integration with Reminders/Notifications modules) based on backend-provided events or schedules. | P1 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Fees UI shall present amounts and statuses clearly on mobile, tablet, and desktop, avoiding clutter. | P0 |
| NF2 | All payment-related interactions shall be performed over secure HTTPS connections, and no full card or sensitive payment data shall be stored in the frontend. | P0 |
| NF3 | Payment initiation screens shall clearly show total payable amount and any applicable breakdown before confirming. | P0 |
| NF4 | The module shall handle duplicate actions gracefully (e.g., double-clicking “Pay”) to avoid duplicate submission. | P1 |

## 3. User Stories & Acceptance Criteria

### Story 1: View my child’s fee dues
**As a** parent, **I want** to see which fees are due for my child **so that** I know what to pay and by when.

**Acceptance criteria:**
- [ ] Given I am logged in as a parent and have selected a child, when I open the Fees section, then I see a list of that child’s fee items with amount, due date, and status.
- [ ] Given some fees are overdue, when I view the list, then overdue items are visually highlighted and ordered or grouped to draw attention.
- [ ] Given a fee has been fully paid, when I view the list, then it is clearly marked as paid and does not appear as payable again.

**Requirement IDs:** F1, F2, F7, NF1

---

### Story 2: Pay fees online
**As a** parent, **I want** to pay my child’s fees online **so that** I don’t have to visit the school in person.

**Acceptance criteria:**
- [ ] Given I have outstanding fee items, when I select item(s) to pay and proceed, then I see a summary of the total amount and items covered before confirming.
- [ ] Given I confirm payment, when the payment gateway and SMS API respond with success, then I see a success confirmation in KYC and the relevant fee items are marked as paid.
- [ ] Given the payment fails or is canceled, when I return to KYC, then I see a clear message indicating the failure and the fees remain unpaid.

**Requirement IDs:** F2, F3, F4, NF2, NF3, NF4

---

### Story 3: View invoices and payment history
**As a** parent, **I want** to see invoices and past payments **so that** I can keep my own records.

**Acceptance criteria:**
- [ ] Given I have made past payments, when I open the Invoices/History view, then I see a list of invoices or payment records with dates, amounts, and statuses.
- [ ] Given I select a specific payment, when I open its details, then I see an invoice/receipt view with itemization and transaction reference (as provided by the backend).

**Requirement IDs:** F5, F6, F7, NF1

## 4. Dependencies
- **SMS API fees endpoints**: Provide fee schedules, statuses, payment initiation URLs/tokens, and payment result callbacks.
|- **Payment gateway integration (via SMS API)**: Handles actual payment processing, PCI compliance, and secure handling of payment credentials; KYC must not integrate directly with gateways beyond what SMS API exposes.|
|- **Reminders/Notifications modules**: Display fee reminders and status updates to parents.|

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Payment failures or delays may cause confusion about whether fees are paid. | High | Medium | Clearly display payment status from backend, include transaction references, and instruct parents to contact the school with reference numbers when in doubt. |
| Complex fee structures (discounts, partial payments) may be hard to visualize. | Medium | Medium | Rely on SMS API to compute final payable amounts; keep UI focused on displaying final amounts and plain-language descriptions. |
| Duplicate payments due to repeated clicks or refreshes. | High | Medium | Use disabled states for buttons after initiation, display clear in-progress states, and rely on idempotent backend/payment gateway behavior. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If a child has no fee items yet or all fees are fully paid, show a clear message like “No outstanding fees” and optionally show history only.
- **Invalid input:** If an invalid fee item or combination is requested (e.g., already paid, invalid ID), show an error message and reload the updated fee list.
- **Failure/offline:** If fee or payment API calls fail, show appropriate error messages and do not assume payment success until confirmed by SMS API.
- **Other:** Handle cases where a payment is marked as pending by the backend by showing a “Pending” state and avoiding duplicate payment attempts until status is resolved.

## 7. MVP Scope

### In scope
- Viewing fee dues and statuses for each child.
- Initiating online payments via SMS API.
- Viewing invoices/receipts and basic payment history.
- Integration with Reminders/Notifications module for fee reminders.

### Out of scope (backlog)
- Advanced payment options like installment planning management in KYC (assumed to be configured in SMS).
- Multi-currency support beyond what SMS API already handles.
- In-app dispute resolution workflows (handled via Complaints module or offline processes).

### Success criteria for MVP
- Significant reduction in in-person fee payments for pilot schools after KYC launch.
- Parents report that fee amounts and payment status are clear and trustworthy in KYC.

