---
title: Payroll — Redesign Plan
type: planned
status: planned
project: platform
module: Payroll
last-updated: 2026-03-29
---

# Payroll — Redesign Plan
### Cursor Implementation Instructions | No code, step-by-step

## Context

The existing payroll module has substantial structure but 5 critical bugs and 20 major gaps. It produces wrong numbers today. This plan fixes the foundation, connects attendance and leave, introduces advance/loan tracking, and redesigns the UX into a single monthly workspace.

> **Before implementing:** Read `docs/product/product-philosophy.md`. Key checks for this module: payroll is a suggestion the system calculates — admin reviews every payslip before finalizing. Individual amendment must always be possible. Staff must be able to see their own payslip. No silent deductions.

Related plans:
- `docs/product/planned/staff-attendance-redesign-plan.md` — attendance pipeline payroll reads from
- `docs/product/planned/leave-management-plan.md` — leave/LOP data payroll reads from

---

## Key Decisions

| Decision | Choice |
|---|---|
| LOP calculation | Auto from finalized attendance + leave data; admin reviews before finalizing |
| PF / ESI / statutory | Fully school-configurable via deduction master — no hardcoded rates or slabs |
| Advance/loan | One-shot disbursement; fixed monthly EMI; auto-deduct until balance = 0; then auto-stop |
| Rounding rule | Configurable per school via payroll settings |
| Working days | Configurable per school via payroll settings |
| Admin review step | Mandatory between Calculate and Finalize — admin must review before lock |
| Individual amendment | Always available per payslip, any time before finalization |
| Staff payslip access | Staff can view and download their own payslips |
| Template / bank details | Editable inline from payroll workspace — no context switching to staff management |
| Payroll history | Full month/year navigation to past finalized runs |

---

## Part 1 — Critical Bug Fixes

### Fix 1.1 — Percentage calculation logic in `_calculate_component_amount`

The current condition `'Deduction' not in str(component)` checks the string representation of the dict — it does not work. Replace with an explicit `component_type` parameter passed into the function so percentage base is determined correctly:
- Earning percentage → base is `gross_salary`
- Deduction percentage → base is determined by `calculation_base` field (`Basic`, `Gross`, or a specific earning's amount)

---

### Fix 1.2 — PF / ESI employer contribution calculation

Remove hardcoded PF and ESI employer multipliers. Instead:
- Each deduction master record has `employer_contribution_rate` (new field — see Step 2.3)
- Employer contribution = `base × employer_contribution_rate / 100`
- If `employer_contribution_rate` is null → no employer contribution for that deduction
- School configures their own rates — system enforces whatever school sets, no government slabs hardcoded

---

### Fix 1.3 — `finalized_by` wrong FK

`PayrollRun.finalized_by` currently references `internal_employees.id`. Change to reference the school's dynamic staff table. Use `staff_id` from the auth context of whoever calls the finalize endpoint.

---

### Fix 1.4 — `rounding_rule` hardcoded

Remove the hardcoded `rounding_rule = 'Nearest'` and the TODO comment. Load it from `{school_id}_payroll_settings.rounding_rule` (see Step 2.1). If payroll settings not configured, default to `Nearest` but flag the payroll run with a warning: "Payroll settings not configured — using defaults."

---

### Fix 1.5 — Basic salary name matching

`basic_salary = next((e['amount'] for e in earnings_list if 'basic' in e['name'].lower()), gross_salary)` — hardcoded string match. Replace with: earning master records have an `is_basic` boolean flag (see Step 2.2). The processing function reads the flag, not the name.

---

## Part 2 — Schema Changes

### Step 2.1 — Add `{school_id}_payroll_settings` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `working_days_per_month` | int, default 26 | Used for per-day rate calculation |
| `rounding_rule` | enum: `nearest`, `up`, `down`, default `nearest` | |
| `payroll_cutoff_day` | int, default 25 | Day of month attendance locks for payroll |
| `payslip_visible_to_staff` | boolean, default true | Whether staff can view their own payslip |
| `created_at` | datetime | |
| `updated_at` | datetime | |

One record per school. Admin configures before first payroll run.

---

### Step 2.2 — Update `{school_id}_earning_masters` table

Add:

| Column | Type | Notes |
|---|---|---|
| `is_basic` | boolean, default false | Marks this earning as Basic Salary — used as deduction base |

Only one earning per school should have `is_basic = true`. Enforce this at the API layer.

---

### Step 2.3 — Update `{school_id}_deduction_masters` table

Add:

| Column | Type | Notes |
|---|---|---|
| `employer_contribution_rate` | decimal(5,2), nullable | Employer's contribution rate %; null = no employer contribution |
| `is_advance_deduction` | boolean, default false | True if this deduction slot is reserved for advance repayment — managed by advance module, not template directly |

---

### Step 2.4 — Update `{school_id}_payroll_runs` table

Add:

| Column | Type | Notes |
|---|---|---|
| `notes` | text, nullable | Admin notes for the run |
| `reviewed_by` | int, FK to staff, nullable | Who reviewed before finalizing |
| `reviewed_at` | datetime, nullable | |

Fix: Change `finalized_by` FK from `internal_employees.id` to reference school staff (no FK constraint — store as plain int with school-scoped validation).

---

### Step 2.5 — Update `{school_id}_payslips` table

Add:

| Column | Type | Notes |
|---|---|---|
| `working_days` | int | Working days in this month (from payroll settings) |
| `paid_days` | decimal(5,1) | working_days − lop_days |
| `lop_days` | decimal(5,1), default 0 | Total LOP days this month |
| `lop_amount` | decimal(10,2), default 0 | Rupee deduction for LOP |
| `lop_breakdown` | json, nullable | Array of `{date, reason}` for each LOP day — transparency for admin |
| `advance_deduction` | decimal(10,2), default 0 | Total advance deducted this month |
| `is_amended` | boolean, default false | True if admin changed anything after system calculated |
| `amended_by` | int, nullable | Staff ID of admin who amended |
| `amendment_notes` | text, nullable | Why it was amended |
| `status` | enum: `draft`, `reviewed`, `finalized`, default `draft` | Per-payslip status |

---

### Step 2.6 — Create `{school_id}_staff_advances` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `staff_id` | int, FK | |
| `total_amount` | decimal(10,2) | Full advance disbursed |
| `monthly_deduction` | decimal(10,2) | Fixed EMI per month |
| `disbursed_date` | date | |
| `disbursed_by` | int, FK to staff | Admin who approved disbursement |
| `total_repaid` | decimal(10,2), default 0 | Running total of repaid amount |
| `remaining_balance` | decimal(10,2) | `total_amount − total_repaid`; updated each payroll run |
| `status` | enum: `active`, `fully_repaid`, `cancelled` | |
| `notes` | text, nullable | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

When `remaining_balance <= 0` → auto-set `status = fully_repaid`. Payroll will not deduct from fully repaid advances.

---

## Part 3 — Backend Pipeline

### Step 3.1 — Update payroll calculation function

Replace the existing `_calculate_salary_from_template` with a corrected version that:

1. Reads `rounding_rule` and `working_days_per_month` from `payroll_settings`
2. Uses `is_basic` flag to identify basic salary for deduction base
3. Calculates earnings correctly:
   - Fixed → use `value`
   - Percentage of Gross → `gross × rate / 100`
   - Percentage of Basic → `basic × rate / 100`
   - Percentage of specific earning → `that_earning_amount × rate / 100`
4. Calculates deductions using `calculation_base` field, not name matching
5. Calculates employer contributions using `employer_contribution_rate` (no hardcoded multipliers)
6. Applies `cap_amount` and `threshold_amount` from deduction master

---

### Step 3.2 — Add LOP calculation step

After salary template calculation, run LOP calculation for each staff member:

1. Fetch finalized `staff_attendance` records for `(staff_id, month, year)` where `is_finalized = true`
2. Count LOP days:
   - `absent` with no approved leave → 1 LOP day
   - `on_leave` with `is_lop_flagged = true` → 1 LOP day
   - `on_leave` with `is_lop_flagged = false` → 0 LOP days
3. Read early permission monthly summary for the month → add fractional LOP days
4. `lop_amount = (gross_salary / working_days_per_month) × lop_days`
5. `net_salary = net_salary − lop_amount`
6. Build `lop_breakdown` array: one entry per LOP day with date and reason
7. Store `lop_days`, `lop_amount`, `lop_breakdown`, `paid_days` on the payslip

If attendance is not yet finalized for the month → flag the payslip: "Attendance not finalized for this month. LOP may be incomplete." Do not block payroll — allow it to proceed with a warning.

---

### Step 3.3 — Add advance deduction step

After LOP calculation:

1. Fetch all `staff_advances` where `staff_id = X` and `status = active`
2. For each active advance:
   - Deduction this month = `MIN(monthly_deduction, remaining_balance)`
   - Add to `advance_deduction` on payslip
   - Add to deductions list as a line item: "Advance Recovery — ₹X (Balance: ₹Y)"
   - Update `total_repaid += deduction_this_month`
   - Update `remaining_balance -= deduction_this_month`
   - If `remaining_balance <= 0` → set advance `status = fully_repaid`
3. `net_salary = net_salary − advance_deduction`

---

### Step 3.4 — Add payroll run lifecycle: Draft → Reviewed → Finalized

Current lifecycle: Draft → Finalized (no review step).

New lifecycle:
```
Calculate → Draft (system calculated, admin has not reviewed)
    ↓
Admin reviews each payslip, amends if needed
    ↓
Mark run as Reviewed (admin confirms they have checked it)
    ↓
Finalize (locked — no further changes)
```

Add `PUT /payroll/{payroll_run_id}/review` endpoint — marks run as `Reviewed`, sets `reviewed_by` and `reviewed_at`.

Finalize endpoint must reject if `status != Reviewed` — admin cannot skip the review step.

---

## Part 4 — New API Endpoints

### Step 4.1 — Payroll Settings

- `GET /payroll/settings` — fetch school payroll settings
- `PUT /payroll/settings` — update payroll settings
- Return a warning in `GET /payroll/runs/{month}/{year}` response if payroll settings are not configured

---

### Step 4.2 — Staff Advances

- `GET /payroll/advances` — list all advances with filters: `staff_id`, `status`
- `POST /payroll/advances` — disburse new advance
- `PUT /payroll/advances/{id}` — admin amendment (adjust monthly deduction, add notes)
- `PUT /payroll/advances/{id}/cancel` — cancel an advance before it's fully repaid
- `GET /payroll/advances/staff/{staff_id}` — all advances for one staff member

---

### Step 4.3 — Individual Payslip

- `GET /payroll/payslips/{id}` — fetch single payslip with full breakdown
- `PUT /payroll/payslips/{id}` — admin amendment; sets `is_amended = true`, records `amended_by` and `amendment_notes`; only allowed when run status is `Draft` or `Reviewed`
- `GET /payroll/staff/{staff_id}/payslips` — staff views their own payslip history (all finalized runs)
- `GET /payroll/payslips/{id}/pdf` — generate payslip PDF for download

---

### Step 4.4 — Payroll Review endpoint

- `PUT /payroll/{payroll_run_id}/review` — mark run as Reviewed; sets `reviewed_by`, `reviewed_at`, updates all individual payslip statuses to `reviewed`

---

### Step 4.5 — Inline staff profile updates from payroll context

These endpoints allow admin to update salary template and bank details without leaving the payroll screen. They write to the staff profile directly but are accessible from the payroll module.

- `PUT /payroll/staff/{staff_id}/template` — assign a new salary template to a staff member; triggers payslip recalculation for current draft run
- `PUT /payroll/staff/{staff_id}/bank-details` — update bank account, IFSC, bank name, branch

---

### Step 4.6 — Payroll History

- `GET /payroll/runs` — list all payroll runs (all months) with status, totals; supports pagination
- `GET /payroll/runs/{month}/{year}` — existing endpoint, no change needed

---

## Part 5 — Flutter UX Redesign

### Step 5.1 — Payroll Settings screen (new, in Payroll module)

Location: Payroll → Settings tab

Contents:
- Working days per month (numeric input)
- Rounding rule (dropdown: Nearest / Up / Down)
- Payroll cutoff day (day of month, numeric)
- Payslip visible to staff (toggle)
- Save button

Show a prominent banner on the payroll home screen if settings are not configured: "Payroll settings not configured. Please set up before running payroll."

---

### Step 5.1b — Salary Templates management screen (new, in Payroll module)

Location: Payroll → Salary Templates tab

**Templates list:**
- List of all salary templates (name, number of components, number of assigned staff)
- Add / Edit / Deactivate template

**Template detail — two tabs:**

*Components tab:*
- Earnings breakdown: each earning component (name, type: fixed/percentage, value/rate, `is_basic` flag)
- Deductions breakdown: each deduction (name, calculation base, rate, cap, employer contribution rate)
- Edit form for each component

*Assigned Staff tab:*
- List of all staff currently on this template
- Department filter
- "Assign Staff" button → staff picker (individual or by department) → confirm
- Remove individual staff from this template
- "Bulk Reassign" — select staff → move to another template → confirm
- On reassignment during an open draft payroll run: flag affected payslips — "Template changed — recalculate?" — do not silently recalculate (admin may have already amended those payslips)

---

### Step 5.2 — Payroll Home screen — History + Month navigation

Replace the current single-month view with a proper home screen:

- Month/year navigation: prev/next arrows + month-year display
- Current month is default on load
- Each month shows: status chip (Not Run / Draft / Reviewed / Finalized), total net, employee count
- Scrollable list of past runs below the current month
- Tap any month → opens the Payroll Workspace for that month

---

### Step 5.3 — Payroll Workspace screen (redesign of current payroll view)

This is the main screen for a specific month's payroll. Single workspace, no context switching.

**Header:**
- Month/year, status chip, summary (total gross, total deductions, total net, LOP total, advance total)
- "Calculate" button (if not yet calculated or to recalculate Draft)
- "Mark as Reviewed" button (available when status = Draft, all payslips checked)
- "Finalize" button (only available when status = Reviewed)

**Warning banners (if applicable):**
- "Attendance not finalized for X staff — LOP may be incomplete"
- "Payroll settings not configured — using defaults"
- "X staff have no salary template assigned"
- "X active advances being deducted this month"

**Staff payslip list:**
- Each row: staff name, department, gross, LOP days, advance deduction, net salary, status chip, amended badge
- Filter: All | Flagged | Amended | No Template
- Tap a row → Staff Payslip Drawer (Step 5.4)
- Rows with `is_amended = true` show a subtle "Amended" badge

---

### Step 5.4 — Staff Payslip Drawer (new)

Triggered by tapping a staff row in the workspace. Renders as a side drawer or bottom sheet.

**Contents:**
- Staff name, designation, department
- Pay period: working days, paid days, LOP days
- Earnings breakdown (line by line)
- Deductions breakdown (line by line, including advance recovery with remaining balance shown)
- Employer contributions section
- Net salary (prominent)
- LOP breakdown: list of each LOP day with reason (from `lop_breakdown` json)
- Advance section: active advance, monthly deduction, remaining balance after this month

**Actions:**
- "Amend Payslip" button — opens inline amendment form:
  - Editable earnings and deductions
  - Amendment notes (required)
  - Submit → sets `is_amended = true`
  - Confirmation dialog: "This will override the system calculation. The amendment will be logged in the audit trail."
- "Change Template" button — opens template selector; on confirm, recalculates this payslip only
- "Update Bank Details" button — inline bank detail form
- Both actions call the new inline endpoints (Step 4.5)

---

### Step 5.5 — Staff Advances screen (new)

Location: Payroll → Advances tab

**Overview:**
- Summary: total active advances, total monthly deduction committed, total disbursed YTD
- List of all advances: staff name, total amount, monthly deduction, remaining balance, status chip
- Filter: Active | Fully Repaid | Cancelled

**New Advance button:**
- Form: staff selector, total amount, monthly deduction (auto-calculates months to repay), disbursed date, notes
- Submit → creates advance; next payroll run picks it up automatically

**Tap advance row:**
- Detail view: full repayment history (which months deducted how much)
- Edit monthly deduction
- Cancel advance

---

### Step 5.6 — Staff: My Payslips screen (SMS app)

Location: My Profile → Payslips tab

- List of all finalized payslips by month
- Each item: month/year, gross, deductions, net
- Tap → full payslip breakdown (same detail as admin drawer but read-only)
- Download button → calls PDF endpoint

Only visible if `payslip_visible_to_staff = true` in payroll settings.

---

## Part 6 — Implementation Order

Run in sequence. Each step is independently testable.

```
1.  Fix _calculate_component_amount — percentage logic (Fix 1.1)
2.  Fix PF/ESI — remove hardcoded multipliers (Fix 1.2)
3.  Fix finalized_by FK (Fix 1.3)
4.  Fix rounding_rule hardcode (Fix 1.4)
5.  Add is_basic flag to earning_master (Fix 1.5)
6.  DB migrations — payroll_settings, staff_advances; update payroll_runs, payslips, earning_masters, deduction_masters (Steps 2.1 → 2.6)
7.  Payroll Settings API (Step 4.1)
8.  Staff Advances API (Step 4.2)
9.  Updated payroll calculation with correct component logic (Step 3.1)
10. LOP calculation step (Step 3.2)
11. Advance deduction step (Step 3.3)
12. Payroll run lifecycle — add Review step (Step 3.4 + Step 4.4)
13. Individual payslip endpoints — get, amend, staff view, PDF (Step 4.3)
14. Inline staff profile update endpoints (Step 4.5)
15. Payroll history endpoint (Step 4.6)
16. Payroll Settings screen (Step 5.1)
17. Salary Templates management screen with Assigned Staff tab (Step 5.1b)
18. Payroll Home — history + month navigation (Step 5.2)
19. Payroll Workspace redesign (Step 5.3)
20. Staff Payslip Drawer with amendment + inline edits (Step 5.4)
21. Staff Advances screen (Step 5.5)
22. Staff My Payslips screen in app (Step 5.6)
```

---

## Gap Resolution Map

| Gap # | Description | Resolved by |
|---|---|---|
| 1 | LOP not calculated | Steps 3.2, 2.5 |
| 2 | Percentage calculation broken | Fix 1.1, Step 3.1 |
| 3 | PF/ESI rates wrong | Fix 1.2, Step 2.3 |
| 4 | Rounding rule hardcoded | Fix 1.4, Step 2.1 |
| 5 | finalized_by wrong FK | Fix 1.3, Step 2.4 |
| 6 | No working days config | Step 2.1 |
| 7 | No attendance integration | Step 3.2 |
| 8 | No leave integration | Step 3.2 |
| 9 | No admin review step | Step 3.4, 4.4, 5.3 |
| 10 | No individual payslip amendment | Step 4.3, 5.4 |
| 11 | Staff can't see payslip | Step 4.3, 5.6 |
| 12 | No payroll settings | Step 2.1, 4.1, 5.1 |
| 13 | Payslip model missing fields | Step 2.5 |
| 14 | PayrollRun missing review fields | Step 2.4 |
| 15 | No payslip PDF | Step 4.3 |
| 16 | No statutory compliance reports | Out of scope — Phase 2 |
| 17 | No advance/loan model | Steps 2.6, 3.3, 4.2, 5.5 |
| 18 | UX fragmented | Step 5.3, 5.4 |
| 19 | No payroll history | Step 4.6, 5.2 |
| 20 | Template/bank details locked behind staff management | Step 4.5, 5.4 |

---

## Out of Scope (Phase 2)

- Statutory compliance reports (PF deposit report, ESI filing, Form 16)
- Bulk bank transfer export (salary disbursement file)
- Multi-level payroll approval
- Arrears calculation
- Bonus / variable pay automation
