---
title: Fees UI UX Improvement Brief
type: spec
project: kyc
module: Fees
last-updated: 2026-03-23
---

> Auto-promoted to testing on 2026-03-23 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

# Fees UI — UX improvement brief

**Goal:** Reduce perceived “spider web” (fewer screens/clicks), clearer hierarchy, more appealing layout. Audience: parents (KYC Fees).

---

## 1. Reduce clicks and screens

| Change | Rationale |
|--------|-----------|
| **Tabs on Fees home** | Add "Dues" and "Invoices & history" as tabs on the same screen. User stays on one screen to switch between dues and history instead of pushing to a separate route. Fewer back-and-forth navigations. |
| **Sticky Pay bar** | When at least one fee is selected, show a sticky bottom bar with total + "Pay selected" so the main action is always visible without scrolling. |
| **Optional: "Select all"** | When there are multiple outstanding items, offer a single "Select all" control so user can pay all in one go without tapping each checkbox. |

**Keep as-is:** Payment summary (sheet) before redirect — needed for confirmation. Payment result screen — needed after gateway return. Invoice detail (push) — needed for receipt content.

---

## 2. Clearer visual hierarchy

| Area | Improvement |
|------|-------------|
| **Summary card** | Make total due the dominant element: larger typography (e.g. headlineSmall or titleLarge for amount). Optional: small icon (e.g. account_balance_outlined) or accent border. Keep compact. |
| **Section label** | Add a short section title above the list (e.g. "Outstanding fees") so the list is scannable. |
| **Fee rows** | Slightly increase spacing between rows. Ensure amount is visually strong (font weight). Status chip remains but doesn’t compete with amount. |
| **Payment sheet** | Add a drag handle at the top of the bottom sheet. Make "Total payable" more prominent (e.g. one line with large amount). |

---

## 3. Layout and polish

- **Spacing:** Use consistent vertical rhythm (e.g. 16px between sections, 12px between list items). Adequate horizontal padding (16–24px).
- **Invoices tab:** Reuse the same list style as payment history (date, amount, status); tap row opens invoice detail (push). Empty state: "No payments yet" with short supporting text.
- **Tabs:** Material 3 tab bar under the summary (or under app title). Min 48px touch targets for tab labels.

---

## 4. Implementation checklist (for Flutter)

- [ ] Add TabBar to Fees home: "Dues" | "Invoices & history". Dues tab = current content (summary + list + Pay). History tab = payment history list (inline; tap row → push invoice detail). Remove standalone "Invoices & history" link and fees/history route from primary flow (optional: keep route for deep link).
- [ ] Sticky bottom bar when `selectedIds.isNotEmpty`: show total + "Pay selected" (primary button). Bar appears above safe area; content scrolls above it.
- [ ] Optional: "Select all" / "Clear" for outstanding items when count > 1.
- [ ] Summary card: larger amount text; optional icon or accent.
- [ ] Section title "Outstanding fees" above the dues list.
- [ ] Payment summary sheet: drag handle at top; emphasize total payable.
- [ ] Spacing and padding pass on cards and list.

No change to payment result or invoice detail screens beyond any minor spacing/typography tweaks. Preserve accessibility (semantics, 48px targets, non–color-only status).
