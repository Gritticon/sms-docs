---
title: Fees UI UX Design
type: spec
project: kyc
module: Fees
last-updated: 2026-03-23
---

> Auto-promoted to testing on 2026-03-23 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

> **Planned** — Design-time UX guidance for the KYC Fees & Invoices feature. Audience: **parents** (view dues, pay online, view invoices and history per active child). Aligns with [fees-ui-build-outline.md](fees-ui-build-outline.md) and KYC UI expectations. Requirements source: [fees-payments-spec.md](../built/fees-payments-spec.md).

# Fees & Invoices — UX Design

---

## 1. Structure

**Order of screens and blocks (top to bottom):**

1. **Fees home** — Summary (if dues exist) → fee dues list → Pay action → entry to Invoices & history.
2. **Payment flow** — Summary & confirm (dialog or bottom sheet) → redirect to gateway → on return: **Payment result**.
3. **Payment result** — Full screen or dialog: status, message, optional reference, primary/secondary actions.
4. **Invoices & history** — Payment history list (rows: date, amount, status); optional filter row if P1; empty state.
5. **Invoice/Receipt detail** — Header, amount, date, items covered, transaction reference; Back.

**Rationale:** Fees home answers “What do I owe?” and “What can I pay?” before any payment. Summary → list → Pay keeps the flow linear. Payment summary & confirm sit between selection and redirect to avoid accidental gateway launch. Result screen gives clear closure. History and detail support “Show me past payments and receipts” without cluttering the main dues view.

**Optional (wide layouts):** Fees home and Invoices & history can be tabs or side-by-side panels; preserve the same block order within each (summary → list → actions; history list → detail).

---

## 2. Wireframes (layout per screen)

### 2.1 Fees home

- **Top:** Optional **summary card** (e.g. AppCard): total due; optional second line for overdue amount when applicable. Compact; only when there are dues. If no dues, omit or replace with empty-state message.
- **Main:** **Fee dues list** — scrollable list of rows. Each row: fee name, amount, due date, status (chip/label). Overdue items first or visually grouped. Optional **checkbox** on each outstanding row for payment selection. Paid items not selectable.
- **Actions:** Primary button (e.g. “Pay selected” / “Proceed to pay”) at bottom or sticky; enabled only when at least one outstanding item is selected and no payment in progress.
- **Navigation:** Link or tab to **Invoices & history** (e.g. text link or tab label below the list or in app chrome).

No pixel specs; use existing patterns: AppCard at top, list below, primary button at bottom.

### 2.2 Payment summary & confirm

- **Container:** Dialog or bottom sheet (modal).
- **Content:** List of selected fee items with amounts; **total payable** prominent (e.g. bold or larger text). Short redirect copy: “You will be redirected to complete payment securely.”
- **Actions:** Confirm (primary) and Cancel (secondary). Min 48px touch targets. After Confirm: button disabled, spinner or “Processing…” visible; no duplicate submit.

### 2.3 Payment result

- **Container:** Full screen or dialog.
- **Content:** Status (Success / Failed / Pending) with clear message; optional transaction reference when available.
- **Actions:** Primary: “Back to Fees” or “View receipt” (success). Secondary: “Retry” when failed. Pending: explain status will update; no “Pay again” for same item.

### 2.4 Payment history list

- **Top (optional P1):** Filter row (e.g. date range, status) if API supports.
- **Main:** List of rows: date, amount, status (success/failed/pending). Tap row → Invoice/Receipt detail.
- **Empty:** Centered or inline message: “No payments yet.”

### 2.5 Invoice/Receipt detail

- **Header:** Title “Invoice” or “Receipt”.
- **Content:** Amount, date, items covered, transaction reference (all from backend; display only).
- **Action:** Back (to history or Fees home).

---

## 3. Components

### 3.1 Fee item row

- **Pattern:** AppCard or ListTile. Content: fee name, amount, due date, **status chip/label** (Paid / Partially paid / Overdue / Upcoming).
- **Checkbox:** Shown only when status is outstanding; min 48px target; theme colors.
- **Status:** Never color-only — use icon + text + theme color (e.g. error for Overdue, primary for Paid). Semantic labels for screen readers.

### 3.2 Summary card

- **Content:** Total due; optional Overdue line when applicable. Compact (single card or one/two lines).
- **Visibility:** Shown only when there are dues; hide or replace with empty message when no outstanding fees.

### 3.3 Status chips/labels

- **States:** Paid, Partially paid, Overdue, Upcoming (dues list); Success, Failed, Pending (payment result / history).
- **UX:** Icon + text + color from theme; semantic labels (e.g. “Overdue”, “Paid”, “Pending”). Sufficient contrast; not conveyed by color alone.

### 3.4 Payment summary content

- **Layout:** List of selected items + amounts; total payable prominent; short redirect copy.
- **Actions:** Confirm (primary), Cancel (secondary). Min 48×48px touch targets. Disable Confirm and show in-progress state after tap.

---

## 4. States

| State | What user sees | What is interactive |
|-------|----------------|----------------------|
| **Loading** | Fees home, history list, invoice detail, payment initiation: overlay or skeleton from request start until response. No fake amounts. | Period selector / nav can stay if appropriate; content area non-interactive until loaded. |
| **Empty (no dues)** | Message “No outstanding fees”; optionally link to payment history. Summary card hidden or replaced. | Link to history (if shown); no Pay button for dues. |
| **Empty (no history)** | “No payments yet” in history list. | Back or link to Fees home. |
| **Error** | AppMessage (or equivalent) with clear, non-technical message; Retry control. | Retry; do not show success or partial payment. |
| **Payment in progress** | Confirm/Pay disabled; “Processing…” or spinner; no duplicate submit. | Cancel only (if allowed by flow); button not tappable. |
| **Success** | Clear success message; optional transaction reference; primary action “Back to Fees” or “View receipt”. | Primary and secondary actions. |
| **Failure** | Failure message; optional reference; Retry and/or Back to Fees. | Retry; Back to Fees. |
| **Pending** | “Payment pending” message; explain status will update; no pay-again for same item. | Back to Fees; no Retry for same payment. |
| **Invalid selection** | Error message (e.g. selection no longer valid). | Reload fee list; user reselects or returns to Fees home. |

**Loading:** One visible loading indicator per logical request; from request start until response (success or error). **Double submit:** Disable button + in-progress state + semantics (announce to screen reader).

---

## 5. Copy & labels

Parent-facing; reuse or refine from build outline §5. Keep consistent across screens.

- **Screen title:** Fees & Invoices (or Fees).
- **Summary:** Total due; Overdue (when applicable).
- **List:** Fee item name, amount, due date; status: Paid / Partially paid / Overdue / Upcoming.
- **Actions:** Pay selected, Proceed to pay; Invoices & history / Payment history.
- **Payment summary:** Total payable; “You will be redirected to complete payment securely.”
- **Payment result:** Payment successful / Payment failed / Payment pending; Transaction reference (when available); Back to Fees; View receipt.
- **Empty:** No outstanding fees; No payments yet.
- **Errors:** Short, non-technical (e.g. “Could not load fees. Please try again.”; “Payment could not be completed.”).

---

## 6. Accessibility

- **Focus order:**  
  - Fees home: summary (if present) → list (each row focusable) → Pay button → Invoices & history link.  
  - Payment flow: summary content → Confirm → Cancel.  
  - Result: message → primary action → secondary action.  
  - History: list → rows → detail; back from detail to list.
- **Status:** Never color-only. Use text + icon + color; semantic labels for “Overdue”, “Paid”, “Upcoming”, “Partially paid”, “Pending”, “Failed” so screen readers announce meaning.
- **Amounts:** Readable typography; sufficient contrast. No sensitive payment data in labels or hints.
- **Touch targets:** Min 48×48px for list rows, buttons, checkboxes, and links.
- **Loading / empty / error:** Screen reader announcements: e.g. “Loading fees”; “No outstanding fees”; “Could not load fees. Tap Retry.”
- **Double submit:** Disable Confirm/Pay after tap; in-progress state visible and announced (e.g. “Processing payment”); prevent duplicate submission.

For full guidance see [kyc-ui-expectations.md](../built/kyc-ui-expectations.md). Use AppButton, AppCard, AppDialog, AppMessage per project rules; theme colors and text styles throughout.

---

## 7. Summary checklist (implementation)

- [ ] **Structure:** Fees home → Payment summary & confirm → Payment result → Invoices & history list → Invoice/Receipt detail; rationale and optional wide layout noted.
- [ ] **Wireframes:** Fees home (summary card → fee list → Pay → Invoices link); Payment summary (list + total + redirect copy, Confirm/Cancel, in-progress); Result (status, message, reference, primary/secondary actions); History list (rows, optional filter, empty state); Invoice detail (header, amount, date, items, reference, Back).
- [ ] **Components:** Fee item row (name, amount, due date, status chip; checkbox when outstanding; theme; not color-only). Summary card (total due, overdue; compact; only when dues). Status chips (four states; icon + text + theme; semantic labels). Payment summary content (list + total + copy; Confirm/Cancel; min 48px targets).
- [ ] **States:** Loading (overlay/skeleton; no fake amounts; from request start). Empty no dues; Empty no history; Error (AppMessage + Retry); Payment in progress (disabled + spinner + semantics); Success/Failure/Pending (message, reference, actions); Invalid selection (error + reload).
- [ ] **Copy & labels:** Aligned with build outline §5; parent-facing and consistent.
- [ ] **Accessibility:** Focus order per screen; status not color-only; amounts readable; 48×48px targets; loading/empty/error announcements; double-submit prevention and semantics.

For acceptance criteria and traceability, see [fees-ui-build-outline.md](fees-ui-build-outline.md).
