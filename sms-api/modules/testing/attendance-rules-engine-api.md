---
title: Attendance Rules Engine — API Spec
type: needs-spec
project: sms-api
module: Staff Attendance
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

# Attendance Rules Engine API

## New Tables

### `{school_id}_AttendanceRule`

| Column | Type | Description |
|---|---|---|
| id | Integer PK | |
| name | String | Rule label |
| description | Text (nullable) | Optional explanation |
| trigger | Enum | See triggers below |
| threshold | Integer | Number of occurrences to fire |
| period | Enum | `daily`, `weekly`, `monthly`, `termly` |
| action | Enum | See actions below |
| action_value | Decimal (nullable) | e.g. 0.5 for half day deduction |
| applies_to | Enum | `school_wide`, `template_specific` |
| template_ids | JSON (nullable) | Template IDs if applies_to = template_specific |
| priority | Integer | Lower = higher priority (default 10) |
| status | Enum | `active`, `inactive` |
| created_at | DateTime | |
| updated_at | DateTime | |

**Trigger Enum:** `late_arrival`, `early_exit`, `break_overstay`, `absence`, `consecutive_absence`, `no_clock_out`

**Action Enum:** `deduct_leave`, `mark_half_day`, `mark_lop`, `notify_staff`, `notify_manager`, `flag_hr`, `credit_overtime`

---

### `{school_id}_AttendanceRuleEvent`

Log of every rule that fired.

| Column | Type | Description |
|---|---|---|
| id | Integer PK | |
| rule_id | Integer | FK to AttendanceRule |
| staff_id | Integer | FK to staff |
| attendance_id | Integer | FK to staff_attendance record |
| trigger_date | Date | Date the trigger event occurred |
| trigger_count | Integer | How many times trigger was counted in the period |
| action_taken | Enum | Action that was executed |
| action_value | Decimal (nullable) | Value used |
| processed_at | DateTime | When the job processed this |
| overridden_by | Integer (nullable) | Staff ID of admin who manually overrode |
| override_reason | Text (nullable) | |

---

## Endpoints

### Rules CRUD

| Method | Endpoint | Description |
|---|---|---|
| GET | `/attendance-rules/` | List all rules |
| POST | `/attendance-rules/` | Create rule |
| GET | `/attendance-rules/{id}` | Get single rule |
| PUT | `/attendance-rules/{id}` | Update rule |
| DELETE | `/attendance-rules/{id}` | Delete rule |
| PATCH | `/attendance-rules/{id}/toggle` | Activate / deactivate |

### Rule Events (Trigger Log)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/attendance-rules/events` | List rule trigger events (filterable) |
| POST | `/attendance-rules/events/{id}/override` | Admin manually overrides a fired rule |

### Manual Processing (admin tool)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/attendance-rules/process` | Manually trigger rule evaluation for a date range |

---

## Background Job

A scheduled job runs to evaluate rules. It does not run per event — it runs on a schedule to batch-process.

**Recommended schedule:**
- Daily job: evaluates daily/weekly trigger counts at end of day
- Monthly job: evaluates monthly trigger counts on last working day of month

**Job logic:**
1. For each active school, load all active rules
2. For each rule, query staff attendance records in the period
3. Count trigger occurrences per staff member
4. If count >= threshold → fire action → write to `AttendanceRuleEvent`
5. Actions that modify attendance status → update `StaffAttendance.status`
6. Notification actions → push via FCM

---

## Conflict Handling

When multiple rules fire for the same staff on the same evaluation:
- Sort rules by priority (ascending)
- Execute in order
- Log all fired rules in `AttendanceRuleEvent`
- If two rules try to set conflicting statuses → highest priority wins

---

## Module / Permission

- Module: 6 (Staff Management)
- Submodule: 506 (Attendance Rules) — new submodule ID to be assigned
