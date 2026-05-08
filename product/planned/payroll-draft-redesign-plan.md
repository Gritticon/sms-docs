---
title: Payroll Draft Redesign — Display-First with Inline Breakdowns
type: planned
status: planned
project: platform
module: Payroll
last-updated: 2026-04-12
---

# Payroll Draft Redesign — Design Plan

## Context

The payroll draft is being redesigned from an **auto-deduction engine** into a
**verification-first tool**. The system surfaces all the facts — attendance
history, template-calculated earnings and deductions — and the drafter makes
the final call before anything is locked.

This plan covers the draft experience only. The salary template system,
lifecycle (Draft → Reviewed → Finalized), advances, and payslip PDF
generation are unaffected.

---

## Core Philosophy

> "The system calculates, the drafter decides."

The salary template populates the draft automatically. The attendance summary
is shown for verification. The drafter removes items they disagree with, adds
items the system cannot know (LOP, bonuses, one-off adjustments), and then
finalises. Nothing is auto-deducted without human sign-off.

---

## Key Decisions

| Decision | Choice |
|---|---|
| Template items in draft | Editable? | Remove only — no inline editing |
| Custom items | Drafter can add label + amount | Both earning and deduction |
| LOP | Not auto-calculated | Drafter adds as a manual deduction if applicable |
| Attendance in draft | Summary only | 22P  3A  1L  26 days |
| Remove staff from draft | Yes | Drafter can exclude a staff member |
| Add staff to draft | Yes | Via [ + Add Staff ] at the bottom |
| Template changes | Do not affect the draft | Template → draft is a one-way copy |
| Grand total | Shown at the bottom | Sum of all staff net salaries |

---

## The Draft Table

### Top-Level View

```
From: [01 Apr 2026]   To: [30 Apr 2026]               [Generate Draft]

Emp ID | Staff Name   | Attendance        | Earnings          | Deductions        | Total    | ×
───────┼──────────────┼───────────────────┼───────────────────┼───────────────────┼──────────┼───
EMP001 | Ravi Kumar   | 22P  3A  1L       | Basic   ₹18,000   | PF       ₹ 2,160  |          |
       |              | 26 working days   | HRA     ₹ 7,200   | ESI      ₹   213  |          |
       |              |                   | Transport ₹1,500  | Prof Tax ₹   200  | ₹27,127  | ×
       |              |                   | Special ₹ 1,800   | ─────────────────  |          |
       |              |                   | ─────────────────  | Total    ₹ 2,573  |          |
       |              |                   | Total   ₹28,500   | [ + Add ]         |          |
       |              |                   | [ + Add ]         |                   |          |
───────┼──────────────┼───────────────────┼───────────────────┼───────────────────┼──────────┼───
EMP002 | Priya Sharma | 26P  0A  0L       | Basic   ₹22,000   | PF       ₹ 2,640  |          |
       |              | 26 working days   | HRA     ₹ 8,800   | ESI      ₹   248  | ₹29,312  | ×
       |              |                   | Transport ₹1,500  | ─────────────────  |          |
       |              |                   | ─────────────────  | Total    ₹ 2,888  |          |
       |              |                   | Total   ₹32,300   | [ + Add ]         |          |
       |              |                   | [ + Add ]         |                   |          |
───────┴──────────────┴───────────────────┴───────────────────┴───────────────────┴──────────┴───
                                                               Grand Total ₹56,439
                                                               [ + Add Staff ]
```

---

### Column Definitions

| Column | Source | Behaviour |
|---|---|---|
| Emp ID | `staff.employee_id` | Read-only |
| Staff Name | `staff.name` | Read-only |
| Attendance | `staff_attendance` records for the period | Summary counts only. Not expandable. |
| Earnings | Template earnings, copied at draft generation | List of line items. Removable. Custom items addable. |
| Deductions | Template deductions, copied at draft generation | List of line items. Removable. Custom items addable. |
| Total | `gross_salary − total_deductions` | Recalculates live on every add/remove |
| × | Remove button | Removes the staff from this draft entirely |

---

### Attendance Summary Format

Counts pulled from `{school_id}_staff_attendance` for the draft period.

```
{days_present}P  {days_absent}A  {days_on_leave}L  {total_working_days} days
```

Example: `22P  3A  1L  26 days`

Statuses mapped:
- **P (Present)** — status `present` or `late`
- **A (Absent)** — status `absent`
- **L (Leave)** — status `on_leave`
- **Holiday** — status `holiday` (not shown in count, excluded from total)

The drafter uses this column to decide whether to add a deduction (e.g. LOP
for the 3 absent days) or treat them as approved leave. The system does not
decide.

---

### Earnings Breakdown Cell

Each item is one row:
```
Basic Salary      ₹ 18,000   [ × ]
HRA               ₹  7,200   [ × ]
Transport Allow.  ₹  1,500   [ × ]
──────────────────────────────────
Total             ₹ 26,700
                  [ + Add ]
```

- Items from the template are pre-populated at draft generation.
- **[ × ]** on any item removes it from this payslip only. Template is unchanged.
- **[ + Add ]** opens an input: `Label [__________]  Amount [________]  [✓ Add]`
- Added items are marked `is_from_template: false` internally.
- Total recalculates immediately after every change.

---

### Deductions Breakdown Cell

Same structure as Earnings. Examples of what the drafter might add:
```
PF               ₹ 2,160   [ × ]   ← from template
ESI              ₹   213   [ × ]   ← from template
──────────────────────────────────
LOP 3 days       ₹   846   [ × ]   ← drafter added
──────────────────────────────────
Total            ₹ 3,219
                 [ + Add ]
```

---

### [ + Add Staff ] Footer

Opens a searchable list of all active staff not currently in the draft.
Selecting a staff member:
1. Loads their assigned salary template
2. Generates earnings and deductions line items from the template
3. Queries attendance for the draft period → populates attendance summary
4. Adds a new payslip row to the draft

---

## Data Model Changes

### Payslip — New Fields

Add attendance summary fields to the `{school_id}_payslips` table:

| Field | Type | Notes |
|---|---|---|
| `days_present` | int, default 0 | Status `present` or `late` |
| `days_absent` | int, default 0 | Status `absent` |
| `days_on_leave` | int, default 0 | Status `on_leave` |
| `days_holiday` | int, default 0 | Status `holiday` |

Keep existing `lop_days`, `lop_amount`, `lop_breakdown` columns.
They are set to 0 at draft generation — not auto-calculated.

### Earnings and Deductions JSON Format

Each item in the `earnings` and `deductions` JSON arrays:

```json
{
  "id": "e_1",
  "label": "Basic Salary",
  "amount": 18000.0,
  "is_from_template": true
}
```

| Field | Notes |
|---|---|
| `id` | Template items: `e_{n}` / `d_{n}`. Custom items: `e_custom_{uuid8}` / `d_custom_{uuid8}` |
| `label` | Display name shown in the cell |
| `amount` | Float — always positive |
| `is_from_template` | `true` = came from template at generation. `false` = drafter-added |

---

## Backend API Changes

### Remove Auto-LOP Calculation

In `payroll_service.py → calculate_payroll()`:

Remove the entire Step 40 block:
- Attendance query for LOP detection
- Loop checking `is_lop_flagged` and `early_permission_monthly_summary`
- `lop_days`, `lop_amount`, `lop_breakdown` auto-calculation
- `net_after_lop` computation

Replace with:
```
lop_days    = 0.0
lop_amount  = 0.0
lop_breakdown = []
net_salary  = gross_salary − total_deductions
paid_days   = working_days_per_month
```

### Attendance Summary in Draft Generation

In `calculate_payroll()`, after salary calculation per staff:

Query `{school_id}_staff_attendance` for the period → count statuses →
write `days_present`, `days_absent`, `days_on_leave`, `days_holiday` onto
the payslip record.

### New Endpoints

| Method | Path | Purpose |
|---|---|---|
| `DELETE` | `/payroll/runs/{run_id}/payslips/{payslip_id}` | Remove staff from draft |
| `POST` | `/payroll/runs/{run_id}/payslips` | Add staff to draft. Body: `{staff_id}` |
| `POST` | `/payroll/payslips/{payslip_id}/line-items` | Add earning or deduction. Body: `{item_type, label, amount}` |
| `DELETE` | `/payroll/payslips/{payslip_id}/line-items/{line_id}` | Remove a line item by its id |

After every add/remove operation, recalculate:
- `gross_salary`, `total_deductions`, `net_salary` on the payslip
- `total_employees`, `total_gross`, `total_deductions`, `total_net` on the run

All four endpoints are Draft-only (or Draft + Reviewed for line items).
Finalized runs cannot be modified.

---

## Flutter UI Changes

### payslip_model.dart

- Add `daysPresent`, `daysAbsent`, `daysOnLeave`, `daysHoliday` fields
- Add `attendanceSummary` getter:
  `'{daysPresent}P  {daysAbsent}A  {daysOnLeave}L  $total days'`
- Add `PayslipLineItem` class: `{id, label, amount, isFromTemplate}`
- Change `earningItems` and `deductionItems` from raw JSON to
  `List<PayslipLineItem>`

### payroll_repository.dart

Add 4 new methods:
- `removePayslipFromRun(runId, payslipId)`
- `addStaffToRun(runId, staffId) → Payslip`
- `addLineItem(payslipId, itemType, label, amount) → Payslip`
- `removeLineItem(payslipId, lineId) → Payslip`

### payroll_workspace_view.dart

Replace the existing payslip list with the new inline-breakdown table:

- Each row renders Earnings and Deductions as vertical item lists with totals
- [ + Add ] per cell opens a label + amount dialog
- [ × ] per item calls `removeLineItem` → patches the single payslip in state
- [ × ] per row calls `removePayslipFromRun` → reloads
- [ + Add Staff ] footer opens staff picker → calls `addStaffToRun` → reloads
- Grand Total footer row: sum of all `payslip.netSalary`
- After `addLineItem` / `removeLineItem`: patch the single payslip in `_payslips`
  list (do NOT do a full reload — only patch the changed row)

---

## What Is NOT Changing

| Component | Status |
|---|---|
| Salary template models and service | Unchanged |
| Draft → Reviewed → Finalized lifecycle | Unchanged |
| Staff advance deductions | Unchanged |
| Payslip amendment trail | Unchanged |
| Payslip PDF generation | Unchanged |
| Payroll settings (working days, rounding) | Unchanged |
| Leave management module | Unchanged |

---

## Implementation Order

Run in sequence. Each step is independently testable.

```
1.  Add attendance summary columns to Payslip model
    (days_present, days_absent, days_on_leave, days_holiday)

2.  Update earnings/deductions JSON format to include id + is_from_template
    in calculate_payroll() — ensure all existing payslip creation uses new format

3.  Remove auto-LOP block from calculate_payroll()
    Set lop_days = 0, lop_amount = 0, net = gross − deductions

4.  Add attendance summary query in calculate_payroll() per staff

5.  Add _recalculate_run_totals() helper to PayrollService

6.  Add 4 new API routes (remove staff, add staff, add line item, remove line item)

7.  Update PayslipLineItem and payslip_model.dart in Flutter

8.  Add 4 new repository methods in payroll_repository.dart

9.  Redesign payroll_workspace_view.dart with inline breakdown table

10. Test full flow:
    Generate draft → verify attendance summary populated
    → remove a line item → verify total updates
    → add a custom deduction → verify total updates
    → remove a staff → verify grand total updates
    → add a staff → verify template items populated
    → finalise → verify payslips locked
```

---

## Out of Scope (Future)

- Attendance history drill-down per staff within the draft
- LOP auto-suggestion based on absent days (show a suggestion, drafter accepts)
- Bulk add/remove of line items across all staff
- Export draft to Excel for offline review
