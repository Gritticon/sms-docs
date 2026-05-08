---
title: Leave Management — Design Plan
type: planned
status: planned
project: platform
module: Leave Management
last-updated: 2026-03-29
---

# Leave Management — Design Plan
### Cursor Implementation Instructions | No code, step-by-step

## Context

Leave management is a new module connecting three systems: **staff app** (request/approval), **attendance** (auto-update on approval), and **payroll** (LOP calculation at month end).

Architecture follows Option A — module-specific rules tables, connected by pipeline.

> **Before implementing:** Read `docs/product/product-philosophy.md`. Every screen, flow, and decision in this plan must pass the design test defined there. Key checks for this module: approval flows must be kept (chit culture), system calculates LOP as a suggestion admin reviews before payroll finalizes, rejected leave correction is always manual (admin in control), zero balance leave is allowed with flagging not blocking.

### Core Principles
- System assists operations, never restricts them — manual fallback available at every step
- Admin can always amend any record regardless of system state
- Auto-calculate everything, surface for admin review before finalizing
- Approved leave → attendance auto-updated via existing attendance_events pipeline
- Rejected leave → admin manually corrects attendance (system does not auto-revert)
- Zero balance leave → allowed, auto-flagged as LOP

---

## Key Decisions

| Decision | Choice |
|---|---|
| Leave types | School configurable — not hardcoded |
| Leave balance | Auto-allocated from leave type quota; manual override as fallback |
| Early permission | Both late-in and early-out, tracked in hours |
| Early permission handling | School configures: deduct from leave balance OR treat as LOP |
| Approver selection | Open — any staff; last approver pre-filled as first suggestion |
| Approval levels | One level (Phase 1); forward to principal in Phase 2 if needed |
| Attendance integration | Approved leave creates `attendance_event` with `source: leave`, `status: on_leave` |
| Zero balance leave | Allowed — auto-flagged as LOP in payroll |
| Rejected leave attendance | Admin manually corrects — no auto-revert |
| Carry forward | School configurable per leave type |
| Payroll LOP | Fully automatic — admin reviews and amends before finalizing |

---

## Part 1 — Database Schema

### Step 1.1 — Add `on_leave` to attendance status enum

In `{school_id}_staff_attendance` and `{school_id}_attendance_events`, add `on_leave` to the existing status enum alongside `present`, `absent`, `late`, `half_day`, `holiday`, `not_marked`.

Also add `leave` to the `source` enum in `{school_id}_attendance_events` alongside `geofence`, `manual`, `biometric`, `face_scan`.

---

### Step 1.2 — Create `{school_id}_leave_types` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `name` | varchar | e.g. "Casual Leave" |
| `code` | varchar | e.g. "CL" — short identifier |
| `days_per_year` | decimal(5,1) | Default annual quota for all staff |
| `carry_forward` | boolean, default false | Unused balance carries to next year |
| `carry_forward_max_days` | decimal(5,1), nullable | Cap on carried forward days; null = no cap |
| `is_active` | boolean, default true | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

### Step 1.3 — Create `{school_id}_leave_balances` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `staff_id` | int, FK | |
| `leave_type_id` | int, FK | |
| `year` | int | Academic/calendar year |
| `allocated_days` | decimal(5,1) | From leave type quota; admin can override |
| `carried_forward_days` | decimal(5,1), default 0 | From previous year |
| `used_days` | decimal(5,1), default 0 | Updated on leave approval |
| `lop_days` | decimal(5,1), default 0 | Days approved with zero balance |
| `is_manual_override` | boolean, default false | True if admin manually set allocated_days |
| `created_at` | datetime | |
| `updated_at` | datetime | |

Add unique constraint on `(staff_id, leave_type_id, year)`.

`remaining_days` is derived: `allocated_days + carried_forward_days - used_days` — do not store, calculate at query time.

Auto-create balance records:
- When a new leave type is created → create balance for all active staff for current year
- When a new staff member joins → create balance for all active leave types for current year
- At year start → carry forward eligible balances and create new year records

---

### Step 1.4 — Create `{school_id}_leave_requests` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `staff_id` | int, FK | staff making the request |
| `leave_type_id` | int, FK | |
| `from_date` | date | |
| `to_date` | date | |
| `total_days` | decimal(5,1) | calculated from dates excluding non-working days |
| `reason` | text | |
| `attachments` | json, nullable | array of S3 URLs |
| `status` | enum: `pending`, `approved`, `rejected`, `cancelled` | |
| `is_lop_flagged` | boolean, default false | true if balance was zero at time of approval |
| `requested_to` | int, FK to staff | approver selected by staff |
| `approved_by` | int, FK to staff, nullable | |
| `approval_note` | varchar, nullable | |
| `approved_at` | datetime, nullable | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

Add index on `(staff_id, from_date, to_date)` and `(requested_to, status)`.

---

### Step 1.5 — Create `{school_id}_early_permission_requests` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `staff_id` | int, FK | |
| `date` | date | |
| `permission_type` | enum: `late_in`, `early_out` | |
| `hours` | decimal(4,2) | e.g. 1.5 hours |
| `reason` | text | |
| `status` | enum: `pending`, `approved`, `rejected`, `cancelled` | |
| `requested_to` | int, FK to staff | |
| `approved_by` | int, FK to staff, nullable | |
| `approval_note` | varchar, nullable | |
| `approved_at` | datetime, nullable | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

Add index on `(staff_id, date)`.

---

### Step 1.6 — Create `{school_id}_early_permission_config` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `half_day_hours` | decimal(4,2) | e.g. 2.0 — X hours = half day |
| `full_day_hours` | decimal(4,2) | e.g. 4.0 — Y hours = full day |
| `handling` | enum: `deduct_leave`, `lop` | school-level default; leave type can override |
| `leave_type_id` | int, FK, nullable | if null = global config; if set = per leave type override |
| `created_at` | datetime | |
| `updated_at` | datetime | |

One record per school (global) plus optional per-leave-type overrides.

---

### Step 1.7 — Create `{school_id}_early_permission_monthly_summary` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `staff_id` | int, FK | |
| `month` | int | 1–12 |
| `year` | int | |
| `total_late_in_hours` | decimal(5,2) | Sum of approved late-in hours |
| `total_early_out_hours` | decimal(5,2) | Sum of approved early-out hours |
| `equivalent_days` | decimal(5,1) | Converted from hours using config thresholds |
| `handling_type` | enum: `deduct_leave`, `lop` | From early_permission_config |
| `lop_days` | decimal(5,1) | Days treated as LOP |
| `leave_deducted_days` | decimal(5,1) | Days deducted from leave balance |
| `processed_at` | datetime | |

Written by the early permission monthly processing function (Step 2.6). Read by payroll LOP calculation (Payroll plan Step 3.2).

---

### Step 1.8 — Add `is_lop_flagged` to `{school_id}_staff_attendance`

Add column: `is_lop_flagged` boolean, default false.

Written by the leave approval pipeline (Step 2.5) when creating an `on_leave` attendance record for a zero-balance leave. Read by payroll LOP calculation (Payroll plan Step 3.2) to distinguish valid leave from LOP leave.

This is a cross-module data contract:
- **Producer:** Leave approval pipeline (Step 2.5)
- **Consumer:** Payroll LOP calculation (Payroll plan Step 3.2)
- **Location:** `{school_id}_staff_attendance.is_lop_flagged`

---

## Part 2 — Backend API

### Step 2.1 — Leave Types CRUD

Full CRUD endpoints under `/leave-types/`:
- `GET /leave-types/` — list all active leave types
- `POST /leave-types/` — create new leave type; auto-creates balance records for all active staff
- `PUT /leave-types/{id}` — update leave type
- `DELETE /leave-types/{id}` — soft delete (set `is_active = false`); do not hard delete

---

### Step 2.2 — Leave Balance endpoints

- `GET /leave-balances/` — all staff balances for current year; supports filter by `staff_id`, `leave_type_id`, `year`
- `GET /leave-balances/staff/{staff_id}` — all leave type balances for one staff member
- `PUT /leave-balances/{id}` — admin manual override of `allocated_days`; sets `is_manual_override = true`
- `POST /leave-balances/carry-forward` — internal function triggered at year start; carries forward eligible balances and creates new year records

---

### Step 2.3 — Leave Request endpoints

- `GET /leave-requests/` — list with filters: `staff_id`, `leave_type_id`, `status`, `from_date`, `to_date`
- `GET /leave-requests/staff/{staff_id}` — all requests for a staff member
- `GET /leave-requests/pending-approvals` — all pending requests where `requested_to` = current user
- `POST /leave-requests/` — create leave request; validate dates are not in the past beyond a configurable window; check balance and set `is_lop_flagged = true` if zero
- `PUT /leave-requests/{id}/approve` — approver approves; triggers leave approval pipeline (Step 2.5)
- `PUT /leave-requests/{id}/reject` — approver rejects; no attendance change (admin corrects manually)
- `PUT /leave-requests/{id}/cancel` — staff cancels a request. Behaviour depends on current status:
  - If `pending` → simple cancel, no side effects
  - If `approved` → reverse leave balance (decrement `used_days` or `lop_days` by `total_days`), flag each attendance record created by this leave as exception (`is_exception = true`, `exception_reason = "Approved leave cancelled — review required"`). Do NOT auto-revert attendance status — admin reviews and corrects manually via the attendance review screen. Audit log the cancellation.
- `PUT /leave-requests/{id}` — admin amendment on any record at any status

Last approver suggestion: `GET /leave-requests/staff/{staff_id}/last-approver` — returns `requested_to` from the most recent leave request for that staff member.

---

### Step 2.4 — Early Permission Request endpoints

- `GET /early-permission-requests/` — list with filters: `staff_id`, `status`, `date`, `month/year`
- `POST /early-permission-requests/` — create request
- `PUT /early-permission-requests/{id}/approve` — approve; triggers early permission processing (Step 2.6)
- `PUT /early-permission-requests/{id}/reject` — reject
- `PUT /early-permission-requests/{id}/cancel` — staff cancels
- `PUT /early-permission-requests/{id}` — admin amendment

---

### Step 2.5 — Leave approval pipeline (internal function)

Triggered when a leave request is approved.

1. Fetch leave balance for `(staff_id, leave_type_id, current_year)`
2. Check remaining balance:
   - If `remaining_days >= total_days` → deduct from `used_days`, `is_lop_flagged = false`
   - If `remaining_days < total_days` → deduct what's available, flag excess as `lop_days`, set `is_lop_flagged = true`
3. For each calendar day in `from_date → to_date`:
   - Skip non-working days (use school's shift template working days)
   - Create `attendance_event` with `source: leave`, `event_type: status_override`, `status: on_leave`, `created_by: approver staff_id`
   - Trigger attendance processing function for that `(staff_id, date)` — same pipeline as attendance redesign
4. Audit log the entire transaction

---

### Step 2.6 — Early permission monthly processing (internal function)

Run this when:
- Month changes (same day-boundary trigger as attendance)
- Admin manually triggers it for a staff member

Logic:
1. Fetch all approved early permission requests for `(staff_id, month, year)`
2. Sum total hours per `permission_type`
3. Apply config thresholds:
   - If total hours >= `full_day_hours` → 1 full day
   - If total hours >= `half_day_hours` → 0.5 day
   - Else → no impact
4. Determine handling from `early_permission_config` (leave deduction or LOP):
   - `deduct_leave` → deduct from the staff member's primary leave type balance; if zero balance → treat as LOP
   - `lop` → directly add to LOP days in the payroll calculation for this month
5. Write result to `early_permission_monthly_summary` table (Step 1.7):
   - `staff_id`, `month`, `year`, `total_late_in_hours`, `total_early_out_hours`, `equivalent_days`, `handling_type`, `lop_days`, `leave_deducted_days`, `processed_at`

---

### Step 2.7 — Payroll LOP calculation (update to existing payroll service)

When payroll runs for a month, the LOP calculation reads:

1. Finalized `staff_attendance` records for the month:
   - `absent` with no approved leave → full LOP day
   - `on_leave` with `is_lop_flagged = true` → LOP day (balance was exhausted)
   - `on_leave` with `is_lop_flagged = false` → no LOP (valid leave)
2. Early permission monthly summary for the month
3. Sum total LOP days → `lop_days` on the payslip
4. LOP deduction = `(monthly_gross / working_days_in_month) × lop_days`

Admin sees the full breakdown: which days caused LOP and why. Admin can amend before finalization.

---

## Part 3 — Flutter UX

### Step 3.1 — Staff: My Leaves screen (SMS app)

Location: My Profile → Leaves & Permissions tab

**Balance section (top):**
- Cards per leave type: name, remaining / total days, used days
- LOP-flagged days shown with orange indicator if any

**History tab:**
- List of all leave requests with status chips
- Filter: All | Pending | Approved | Rejected
- Tap a request → detail view with approval note

**New Request button:**
- Floating action button
- Form: Leave type dropdown, from/to date picker (no future beyond configurable window), reason text, attachment upload
- Approver field: auto-fills last approver; searchable dropdown of all active staff
- If balance is zero → show warning banner: "No balance remaining. This will be marked as LOP if approved."
- Submit → status becomes pending

---

### Step 3.2 — Staff: My Early Permissions screen (SMS app)

Location: My Profile → Leaves & Permissions tab (separate section within the same tab as My Leaves — staff sees both leave requests and early permission requests in one place)

**Summary section:**
- Current month: hours used, threshold status (e.g. "1.5 of 4 hours used this month")

**History tab:**
- List of all early permission requests with status
- Filter by month

**New Request button:**
- Form: Date, type (Late In / Early Out), hours, reason
- Approver field: same auto-fill logic as leave requests
- Submit → status becomes pending

---

### Step 3.3 — Approver: Pending Approvals tab (SMS app)

Location: My Profile → Approvals tab

Follows the same tab pattern as other profile tabs (timetable, attendance). No new nav item required — approvals are contextually tied to the staff member who handles them.

**Pending tab (default):**
- List of all pending requests assigned to the current user (both leave and early permission)
- Each item shows: staff name, type, dates/hours, reason
- Tap → detail view
- Approve button (with optional note) + Reject button (note required)
- Approval triggers pipeline immediately
- Undo toast: "Approved — Undo" (4 second window) — same pattern as attendance review

**History tab:**
- All requests this staff member has previously approved or rejected
- Filter: All | Approved | Rejected
- Read-only — for personal reference

**Notifications (required for discoverability):**
- Push notification to approver when a new request is assigned to them — deep links directly to this tab
- Push notification to requesting staff when their request is approved or rejected — includes approval note if rejected

---

### Step 3.3b — Admin: Staff Approvals audit tab (SMS web)

Location: Staff Management → Staff Profile → Approvals tab

Admin view of the same data — all approvals and rejections made by this staff member as an approver. Read-only audit trail.

- Each row: requesting staff name, request type, dates, decision (approved/rejected), decision date, note
- Useful for HR oversight: identifying approvers sitting on pending requests or patterns in rejections
- Filter: All | Approved | Rejected | Pending

---

### Step 3.4 — Admin: Leave Overview screen (SMS web)

Location: Staff Management → Leave

This is a tabbed screen with two tabs:

**Overview tab (default):**
- Summary bar: total pending requests, total LOP-flagged this month, staff with zero balance
- Staff list: each row shows name, department, leave balance summary per type, LOP days this month
- Filter by department, leave type, status
- Tap a staff row → Staff Leave Detail drawer

**Requests tab:**
- All leave requests across all staff (see Step 3.5)

**Staff Leave Detail drawer:**
- All leave requests for this staff (all time, paginated)
- Balance per leave type with manual override option
- Early permission history for current month
- Amendment button on any record

---

### Step 3.5 — Admin: Leave Requests screen (SMS web)

Location: Staff Management → Leave → Requests tab

- All leave requests across all staff
- Filters: status, date range, department, leave type
- Admin can approve, reject, or amend any request regardless of status
- Amendment creates an audit log entry

---

### Step 3.6 — Attendance Setup: Leave Configuration (isolated section)

Extend the Setup section defined in the attendance redesign plan (Step 3.5). Leave configuration lives under Setup → Leave — a sub-section at the same level as Setup → Attendance.

Add two sub-screens:

**Leave Types:**
- List of configured leave types with quota, carry forward, early permission handling
- Add / Edit / Deactivate
- Editing quota prompts: "Apply new quota to existing staff this year? Yes / No"

**Early Permission Config:**
- Global thresholds: X hours = half day, Y hours = full day
- Handling: deduct from leave or LOP
- Per-leave-type overrides (optional)

---

## Part 4 — Implementation Order

Run in sequence. Each step is independently testable.

```
1.  Add on_leave to attendance status enum and leave to source enum (Step 1.1)
2.  DB migrations — leave_types, leave_balances, leave_requests, early_permission_requests, early_permission_config, early_permission_monthly_summary; add is_lop_flagged to staff_attendance (Steps 1.2 → 1.8)
3.  Leave Types CRUD API (Step 2.1)
4.  Leave Balance endpoints (Step 2.2)
5.  Leave Request endpoints — create, list, cancel (Step 2.3, partial)
6.  Early Permission Request endpoints — create, list, cancel (Step 2.4, partial)
7.  Leave approval pipeline (Step 2.5)
8.  Approve/Reject endpoints for leave and early permission (Step 2.3 + 2.4, complete)
9.  Early permission monthly processing (Step 2.6)
10. Payroll LOP calculation update (Step 2.7)
11. Staff: My Leaves screen (Step 3.1)
12. Staff: My Early Permissions screen (Step 3.2)
13. Approver: Pending Approvals screen (Step 3.3)
14. Admin: Leave Overview + Staff Leave Detail drawer (Step 3.4)
15. Admin: Leave Requests screen (Step 3.5)
16. Attendance Setup: Leave Configuration screens (Step 3.6)
```

---

## Integration Map

```
Leave Request approved
    → leave_approval_pipeline()
        → deducts leave_balance
        → creates attendance_events (source: leave, status: on_leave)
        → attendance processing runs → updates staff_attendance

Early permissions accumulate monthly
    → early_permission_monthly_processing()
        → converts hours → days
        → deducts leave_balance or flags as LOP

Month end payroll run
    → reads finalized staff_attendance
    → reads early permission monthly summary
    → calculates total LOP days
    → admin reviews → amends → finalizes
```

---

## Out of Scope (Phase 2 if needed)

- Multi-level approval (coordinator → principal)
- Leave encashment
- Leave application via KYC (parent-side for student leaves is separate)
- Automated leave balance reset notifications
