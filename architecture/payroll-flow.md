# Payroll Flow

## Overview

Two-layer component architecture:

1. **Master catalog** (`{school_id}_earning_masters`, `{school_id}_deduction_masters`) — library of reusable components
2. **Template components** (`{school_id}_earnings`, `{school_id}_deductions`) — per-template line items copied from masters or created directly

Staff are assigned a salary template. Payroll runs calculate each staff member's payslip from their template.

---

## Component Fields

### Earning (master + template component)
| Field | Purpose |
|---|---|
| `name` | Display label |
| `code` | Unique identifier |
| `calculation_type` | `Fixed` or `Percentage` |
| `calculation_base` | `Basic`, `Gross`, or `Earning` (percentage anchor) |
| `value` | Amount (Fixed) or % rate (Percentage) |
| `priority` | Calculation order (lower = first) |
| `description` | Optional notes |
| `status` | `Active` / `Inactive` (master only) |

### Deduction (master + template component)
Same fields as Earning plus:
| Field | Purpose |
|---|---|
| `category` | `Statutory`, `Insurance`, `Loan`, `Other` (master only) |

**Intentionally omitted** (over-engineering for school context):
- `taxability`, `include_in_pf_base`, `include_in_esi_base` — stored but not used in calc
- `cap_amount`, `threshold_amount` — use multiple templates instead
- `employer_contribution_rate` — not used in template-based calc
- `is_advance_deduction`, `impact_on_net_salary` — not used in calc

---

## Payroll Run Lifecycle

```
Draft → Calculated → Finalized
```

### 1. Create Draft Run
`POST /payroll/runs`
- Fields: `month`, `year`, `date_from`, `date_to`
- Status: `Draft`

### 2. Calculate
`POST /payroll/runs/{id}/calculate`

For each active staff with a salary template:
1. Load template earnings + deductions via `get_template_earnings()` / `get_template_deductions()`
2. Resolve legacy fields via coalesce (`calculation_type` ← `type`, `value` ← `percentage`/`amount`)
3. Run `_calculate_salary_from_template(earnings, deductions, basic_salary)`

### 3. Salary Calculation Logic (`_calculate_salary_from_template`)

```
basic = staff.basic_salary (fixed seed, never changes)

# Earnings loop (sorted by priority)
for each earning:
    if Fixed:  amount = value
    if Percentage:
        base = Basic → basic
               Gross → gross (= basic at this point in loop)
               Earning → earnings_by_id[base_earning_id]
    gross += amount

# Deductions loop (sorted by priority)
for each deduction:
    if Fixed:  amount = value
    if Percentage:
        base = Basic → basic
               Gross → final gross (after all earnings)
               Earning → earnings_by_id[base_earning_id]

net = gross - sum(deductions)
```

### 4. Attendance Pro-rata
Attendance days from `{school_id}_attendance` for `date_from..date_to`.
Working days from payroll run settings. Pro-rata applied before deductions.

### 5. Advance Deductions
`{school_id}_advance_deductions` queried per staff for the run month.
Added as separate deduction line items.

### 6. Review / Amend
`PUT /payroll/runs/{id}/payslips/{payslip_id}` — manual override of individual line items.

### 7. Finalize
`POST /payroll/runs/{id}/finalize`
- Status: `Finalized`
- Locked — no further edits

---

## Backward Compatibility

Old template rows stored `type`/`amount`/`percentage` (legacy schema). The `_resolve_earning`/`_resolve_deduction` helpers coalesce:

```python
calc_type = e.calculation_type or (e.type.capitalize() if e.type else "Fixed")
raw_value = e.value if e.value is not None else (
    e.percentage if type == "percentage" else e.amount
)
```

New rows always use `calculation_type` + `value`.

---

## Key Files

| Layer | File |
|---|---|
| Payroll calc | `sms-api/app/services/payroll_service.py` |
| Template CRUD | `sms-api/app/services/salary_template_service.py` |
| Earning master model | `sms-api/app/models/earning_master.py` |
| Deduction master model | `sms-api/app/models/deduction_master.py` |
| Template component models | `sms-api/app/models/earning.py`, `deduction.py` |
| Flutter templates UI | `sms-ui/lib/views/payroll_templates_view.dart` |
| Flutter masters UI | `sms-ui/lib/views/earnings_master_view.dart`, `deduction_master_view.dart` |
| DB migrations | `sms-api/alembic/versions/simplify_earning_deduction_components.py` |
