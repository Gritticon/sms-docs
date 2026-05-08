---
title: Fees UI Build Outline
type: spec
project: kyc
module: Fees
last-updated: 2026-03-23
---

> Auto-promoted to testing on 2026-03-23 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

> **Planned** — Implementation source for the Fees & Invoices UI in KYC. Basis for UX to develop a detailed design. Requirements source: [fees-payments-spec.md](../built/fees-payments-spec.md).

# Fees & Invoices — UI Build Outline (KYC)

Outline for building the Fees & Invoices UI in KYC. Target user: **parents** (view dues, pay online, view invoices and history per child). All fee structures, amounts, and payment processing live in the SMS backend; KYC presents a clear UI to initiate and track payments. No payment data (card details, etc.) in the frontend.

---

## 1. Confirmed Assumptions

| Item | Decision |
|------|----------|
| **Primary user** | Parents. Data and actions are per **active child**; multi-child via active context (child switcher). |
| **Source of requirements** | [fees-payments-spec.md](../built/fees-payments-spec.md) — F1–F8, NF1–NF4, user stories, MVP scope. |
| **Payment flow** | Initiate payment from KYC → SMS API returns URL/token or redirect → parent completes payment in gateway; result returned to SMS and surfaced in KYC. No card/sensitive data in Flutter. |
| **Class/section** | Inferred from active child; user does not select. |
| **Amounts & statuses** | All from SMS API. UI displays only; no calculations or business logic in frontend. |
| **Reminders** | Fee reminders surfaced via Reminders/Notifications module (F8); not a separate Fees sub-screen. |
| **MVP boundary** | In scope: view dues & statuses, initiate payment, confirmation, invoices/receipts, basic payment history. Out of scope: installment planning in KYC, multi-currency beyond API, in-app dispute workflows. |

---

## 2. Entry & Flow Summary

1. **Entry:** User opens Fees (e.g. from KYC nav). Default view = **Fees home** for the **active child**.
2. **Fees home:** Optional summary/total due at top; then **fee dues list** (name, amount, due date, status: paid / partially paid / overdue / upcoming). Overdue items visually highlighted and ordered/grouped first. Pay action (e.g. "Pay selected" or "Proceed to pay") for outstanding items.
3. **Payment flow:** User selects one or more fee items → **Payment summary** (items, breakdown, total) → Confirm → redirect or handoff to SMS API / payment gateway. On return: **Payment result** (success / failure / pending) with clear message and transaction reference when available.
4. **Invoices & history:** Separate section or tab: **Payment history list** (date, amount, status). Tap a record → **Invoice/Receipt detail** (amount, date, items covered, transaction reference). Filters (e.g. date range, status) if backend supports (P1).
5. **Multi-child:** When parent switches active child, Fees home and history reload for the selected child; no separate "select child" on Fees screen.
6. **Duplicate actions:** Pay button disabled after tap; in-progress state visible; idempotent backend behavior assumed (NF4).

---

## 3. Screens & Components (UI Build Order)

### 3.1 Fees home (main)

- **Placement:** Default screen when user opens Fees. Data for active child only.
- **Optional summary card (top):** Total due, overdue amount (if any). Keep compact; can be single AppCard or one line. If no dues, show "No outstanding fees" or hide summary and show empty state for list.
- **Fee dues list:** List of fee items from API: name, amount, due date, status (paid / partially paid / overdue / upcoming). Use status colors from theme (e.g. error for overdue, primary for paid). Overdue items first or visually grouped. Paid items clearly marked and not selectable for payment.
- **Selection (for payment):** Allow selecting one or more **outstanding** items (checkboxes or similar). "Pay selected" or "Proceed to pay" enabled only when at least one outstanding item is selected and no payment in progress.
- **Invoices / History entry:** Link or tab to payment history list (see 3.4). Label e.g. "Invoices & history" or "Payment history".
- **Loading:** From request start until fee list (and summary if used) response. Per project rules: visible loading for every API request.

### 3.2 Payment flow — Summary & confirm

- **Trigger:** User taps "Pay selected" / "Proceed to pay" from Fees home with one or more items selected.
- **Summary screen (or bottom sheet):** List of selected items with amounts; **total payable** prominent. Short copy: "You will be redirected to complete payment securely." Confirm and Cancel actions.
- **On Confirm:** Call SMS API to initiate payment; disable button, show in-progress. API returns redirect URL or token → open in browser/webview or external app per backend contract. No card fields in KYC.
- **Back from gateway:** Handle return URL or callback; request payment status from SMS API. Navigate to Payment result (3.3).

### 3.3 Payment result

- **Trigger:** After payment attempt (success, failure, or pending) when KYC receives or fetches status from SMS API.
- **Content:** Clear status message: Success / Failed / Pending. On success: optional short confirmation text and transaction reference. On failure: message (e.g. "Payment could not be completed") and option to retry or return to Fees home. On pending: explain that status will update; avoid duplicate payment attempts.
- **Actions:** "Back to Fees" or "View receipt" (if success and receipt available).

### 3.4 Payment history list

- **Entry:** From Fees home via "Invoices & history" / "Payment history" link or tab.
- **Content:** List of payment records for active child: date, amount, status (success/failed/pending). Optional filters (date range, status) if API supports (P1).
- **Empty:** "No payments yet" when list is empty.
- **Tap row:** Navigate to Invoice/Receipt detail (3.5).

### 3.5 Invoice / Receipt detail

- **Trigger:** From payment history list (or from success result "View receipt").
- **Content:** Single payment: amount, date, items covered, transaction reference. All fields from backend; no formatting logic beyond display. Printable/shareable if product requires (later).
- **Loading / error:** Loading until detail loaded; error state with retry or back.

### 3.6 Reusable / shared

- **Fee item row:** Used in Fees home list: name, amount, due date, status badge; optional checkbox for selection when status is outstanding. AppCard or ListTile; min 48px touch target.
- **Summary card:** Optional total due / overdue block on Fees home; reuse pattern from current placeholder but data from API.
- **Status chips/labels:** Paid, Partially paid, Overdue, Upcoming — theme colors and semantics.

---

## 4. States to Implement

| State | Where | Behavior |
|-------|--------|----------|
| **Loading** | Fees home, payment history, invoice detail, payment initiation | Show loading indicator from request start until response (success or error). One loading state per logical request. |
| **Empty — no dues** | Fee dues list | "No outstanding fees" (and optionally show link to payment history only). |
| **Empty — no history** | Payment history list | "No payments yet". |
| **Error** | Any API failure | AppMessage (or equivalent) with clear message; Retry where appropriate. Do not assume payment success until confirmed by SMS API. |
| **Payment in progress** | Payment flow (after Confirm) | Disable Pay/Confirm button; show spinner or "Processing…"; prevent duplicate submit. |
| **Success** | Payment result | Success message, optional transaction reference, "Back to Fees" / "View receipt". |
| **Failure** | Payment result | Failure message; option to retry or return to Fees home. |
| **Pending** | Payment result / history | "Pending" state; avoid offering pay again for same item until status resolved. |
| **Invalid selection** | Payment flow | If user somehow has invalid selection (e.g. already paid), show error and reload fee list. |

---

## 5. Copy & Labels (parent-facing)

- Screen title: **Fees & Invoices** (or **Fees**).
- Summary: **Total due**, **Overdue** (when applicable).
- List: Fee item **name**, **amount**, **due date**, **status** (Paid / Partially paid / Overdue / Upcoming).
- Actions: **Pay selected**, **Proceed to pay**; **Invoices & history** / **Payment history**.
- Payment summary: **Total payable**; "You will be redirected to complete payment securely."
- Payment result: **Payment successful** / **Payment failed** / **Payment pending**; **Transaction reference** (when available); **Back to Fees**, **View receipt**.
- Empty: **No outstanding fees**; **No payments yet**.
- Errors: Short, non-technical (e.g. "Could not load fees. Please try again."; "Payment could not be completed.").

---

## 6. Backend Dependencies (high level)

- **Fees list API** — Per active child: fee items/schedules with name, amount, due date, status (paid / partially paid / overdue / upcoming). Optional summary (total due, overdue) from same or separate endpoint.
- **Payment initiation API** — Accept selected fee item IDs; return redirect URL or token for payment gateway. KYC opens URL; no card data in app.
- **Payment result / callback** — SMS API receives gateway callback; KYC fetches payment status (success / failure / pending) and transaction reference after return.
- **Payment history API** — List of payments for active child (date, amount, status); optional filters (date range, status). P1.
- **Invoice/receipt API** — Detail for a single payment: amount, date, items covered, transaction reference.
- **Active-child context** — Same as rest of KYC; fees and history scoped to selected child.

Endpoint shapes and payloads belong in SMS/API docs; KYC consumes and handles loading/empty/error in UI.

---

## 7. UX / Accessibility

- **Focus order:** Summary (if present) → fee list (each row focusable) → Pay action → Invoices/history entry. Payment flow: summary → Confirm → Cancel. Result screen: message → primary action → secondary action.
- **Status:** Use text + color + icon where appropriate; status not conveyed by color alone (WCAG). Labels like "Overdue", "Paid", "Upcoming" announced to screen readers.
- **Amounts:** Clear, readable typography; ensure contrast. No sensitive payment data in labels.
- **Touch targets:** Min 48×48px for list rows, buttons, checkboxes, and links.
- **Loading / empty / error:** Announced to screen readers; loading state clear (e.g. "Loading fees"); empty and error messages concise and actionable.
- **Double submit:** Disable Pay/Confirm after tap; in-progress state visible and announced.

For full guidance see [kyc-ui-expectations.md](../built/kyc-ui-expectations.md). Use AppButton, AppCard, AppDialog, AppMessage per project rules; theme colors and text styles throughout.

---

## 8. Reference

- Requirements, user stories, MVP scope, edge cases: [fees-payments-spec.md](../built/fees-payments-spec.md).
- KYC UI expectations and design system: [kyc-ui-expectations.md](../built/kyc-ui-expectations.md).
- Current placeholder (to be replaced): `kyc/lib/views/finance/finance_views.dart` — FeesView (static list, Summary card, "Proceed to pay (mock)").
