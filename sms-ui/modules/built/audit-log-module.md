---
title: Audit Log Module
type: spec
project: sms-ui
module: Audit Logs
last-updated: 2026-03-21
---

> Auto-promoted to built on 2026-03-21 — implementation detected in codebase.

# Audit Log Module

## Purpose

Provide a centralised, permission-gated view of all write and action events across the SMS platform. Anyone with the Audit Log module permission can see a full trail of what was changed, when, and by whom — across all modules that have audit logging enabled.

This is a read-only module. It does not create data; it surfaces data written by the audit logging framework running in sms-api.

---

## Scope

**In scope:**
- Standalone module in the sms-ui navigation (not embedded inside other modules)
- Filterable list of audit log entries with a detail view
- Backend decorator framework standard — the pattern for wiring any endpoint
- Removing all existing per-module audit drawers and centralising into this module

**Out of scope:**
- Which specific modules and endpoints to wire up (decided separately after module is built)
- Export (CSV, PDF) — planned future addition
- Real-time/live updates — not required for v1

---

## Permission Model

Audit Logs is a new SMS module. Add it per the checklist at `docs/architecture/permissions-adding-modules-checklist.md`.

| Permission | Description |
|---|---|
| View Audit Logs | See the audit log list and detail. No write permission exists — this is a read-only module. |

Anyone with the View permission can see all audit entries across all modules — there is no per-module scoping of the audit log view. If a staff member should not see payroll audit entries, they should not have the Audit Log permission at all.

---

## Backend Framework

The audit logging infrastructure already exists in sms-api. This section defines the **standard** for how it is applied consistently across all endpoints.

### Decorator Usage

```python
@router.post("/staff", response_model=StaffCreateResponse)
@audit_log(module="staff_management", action="create_staff")
async def create_staff(...):
```

The decorator:
- Runs the endpoint first, logs only on success (exceptions are not logged)
- Captures: `school_id`, `module`, `action`, `edited_by_staff_id`, `timestamp`, `payload`
- Stores to S3: `audit/{module}/{school_id}/{year}/{module}_{school_id}_{year}-{month}.json`
- Gracefully skips logging on storage failure — never breaks the main operation

### Action Naming Convention

All action names follow `{verb}_{noun}` in snake_case. Use only the standard verbs below:

| Verb | When to use |
|---|---|
| `create` | A new record is created |
| `update` | An existing record is modified |
| `delete` | A record is deleted |
| `finalize` | A period or record is closed/locked (e.g. payroll finalised) |
| `generate` | A set of records is generated from configuration (e.g. timetable generated) |
| `publish` | Content is made visible to others (e.g. results published, announcement published) |
| `assign` | An assignment is made (e.g. teacher allotted to subject) |
| `approve` | A request or complaint status is moved to approved/resolved |
| `reject` | A request or complaint is rejected |

Examples:
- `create_staff`, `update_staff`, `delete_staff`
- `create_timetable`, `generate_timetable`
- `finalize_payroll`, `update_payroll`
- `publish_results`, `publish_announcement`
- `assign_substitute`

**Read actions** (e.g. `view_payslip`, `view_staff_payroll`) should **not** be logged. Audit logging is for state-changing actions only. Existing read-action decorators should be removed when each module is reviewed.

### Module Name Convention

The `module` parameter in the decorator should match the module's identifier used elsewhere in the system (kebab-case mapping to the module name):

| Module | `module` value |
|---|---|
| Staff Management | `staff_management` |
| Student Management | `student_management` |
| Class Management | `class_management` |
| Schedule Management | `schedule_management` |
| Exam Management | `exam_management` |
| Staff Attendance | `staff_attendance` |
| Payroll | `payroll` |
| Complaints | `complaints` |
| Timetable | `timetable` |
| Class Session | `class_session` |
| School Management | `school_management` |
| Communication Hub | `communication_hub` |

### Custom Payload Extraction

When the default payload (request body) does not capture enough context, use the `extract_payload` callable:

```python
@audit_log(
    module="payroll",
    action="finalize_payroll",
    extract_payload=lambda result, request: [{"period": result.period, "staff_count": result.staff_count}]
)
```

Use this when: the important context is in the response (not the request), or when the default payload is too large and needs trimming.

---

## API

The existing endpoint serves audit log data. The only change needed is adding `limit` and `offset` support for the "load more" pattern.

### Endpoint

```
GET /audit-logs/
```

| Parameter | Type | Description |
|---|---|---|
| `module_id` | int | Filter by module |
| `submodule_id` | int (optional) | Filter by submodule |
| `staff_id` | int (optional) | Filter by staff member who made the change |
| `date` | date (optional) | Filter by specific date |
| `limit` | int (default: 20) | Number of entries to return |
| `offset` | int (default: 0) | Entries to skip (for load more) |

**Response:** Returns entries in reverse chronological order. Payload field is excluded for performance — only included in a future detail endpoint if needed.

---

## UI Module

### Navigation

Audit Logs appears as a standalone item in the sms-ui sidebar navigation, visible only to users with the View permission.

---

### Screen 1 — Audit Log List

The primary view. Displays log entries in reverse chronological order with filter controls at the top.

**Filters (all optional, combinable):**

| Filter | Control | Behaviour |
|---|---|---|
| Module | Dropdown | Shows all modules that have audit logging; default: All |
| Staff Member | Searchable dropdown | Filter entries by who performed the action |
| Date | Date picker (single day) | Filter by a specific date |
| Date Range | From / To date pickers | Filter by a date range |

**List columns:**

| Column | Content |
|---|---|
| Timestamp | Date and time of the action |
| Module | Module name (human-readable, not ID) |
| Action | Human-readable action description (from `description` field) |
| Performed by | Staff name |

**Behaviour:**
- Default load: 20 most recent entries across all modules
- **Load more** button at the bottom — appends the next 20 entries (offset-based)
- Load more is hidden when there are no more entries to load
- Filters reset the list and fetch from offset 0
- Clicking a row opens the detail drawer

---

### Screen 2 — Audit Log Detail (Drawer)

A slide-out drawer that opens when a list entry is tapped. Displays the full context of that single log entry.

**Fields shown:**

| Field | Description |
|---|---|
| Timestamp | Full date and time |
| Module | Module name |
| Submodule | Submodule name (if applicable) |
| Action | Action label (e.g. "Update Staff") |
| Description | Human-readable description generated by sms-api |
| Performed by | Staff name and role |
| Payload | Formatted display of the data that was submitted — shown as a key-value list, not raw JSON |

**Payload display:** Render the payload as a readable key-value list (e.g. "First Name: John → Jane") rather than raw JSON. If the payload is empty or null, show "No payload recorded."

---

## Flutter Architecture

### Model

Replace all existing module-specific audit log models with a single unified model:

```
lib/models/audit_log_model.dart
```

Fields: `id`, `timestamp`, `module`, `submodule`, `action`, `description`, `editedByStaffId`, `editedByStaffName`, `payload` (nullable map)

### Controller

```
lib/controllers/audit_log_controller.dart
```

Single GetX controller managing:
- Filter state (module, staff, date)
- Entry list (with append-on-load-more)
- Drawer open/close state and selected entry
- API calls via `AuditLogRepository`

### Repository

```
lib/repositories/audit_log_repository.dart
```

Wraps the `GET /audit-logs/` endpoint. Accepts filter and pagination parameters.

### Views and Widgets

```
lib/views/audit_log/audit_log_view.dart         # Main list screen
lib/widgets/audit_log/audit_log_list_item.dart  # Single row widget
lib/widgets/audit_log/audit_log_drawer.dart     # Detail drawer
lib/widgets/audit_log/audit_log_filters.dart    # Filter bar widget
```

---

## Migration — Existing Audit Drawers

The following files must be **deleted** when this module is built. They are replaced entirely by the centralised module.

### Files to delete

**Widgets:**
- `lib/widgets/attendance_audit_log_widget.dart`
- `lib/widgets/attendance_audit_log_drawer.dart`
- `lib/widgets/staff_profile_audit_log_drawer.dart`
- `lib/widgets/payroll_audit_log_widget.dart`
- `lib/widgets/payroll_audit_log_drawer.dart`
- `lib/widgets/roles_departments_audit_log_drawer.dart`

**Models:**
- `lib/models/attendance_audit_log_model.dart`
- `lib/models/staff_profile_audit_log_model.dart`
- `lib/models/payroll_audit_log_model.dart`
- `lib/models/roles_departments_audit_log_model.dart`

**Controllers:**
- `lib/controllers/roles_departments_audit_controller.dart`

### Views to update

Remove audit log drawer triggers and references from:
- `lib/views/staff_profiles_view.dart`
- `lib/views/staff_attendance_view.dart`
- `lib/views/student_attendance_view.dart`
- `lib/views/payroll_view.dart`
- `lib/views/roles_departments_view.dart`
- `lib/views/profile_view.dart`

---

## Existing Wired Endpoints — Review Required

94 decorators across 13 route files are already in place. Before this module ships, each must be reviewed against the naming convention above:

| File | Review action |
|---|---|
| `payroll.py` | Remove read-action decorators (`view_staff_payroll`, `view_payroll_run`, `view_payslips`); confirm write actions match convention |
| `timetables.py` | Confirm action names match convention |
| `academic_years.py` | Confirm action names match convention |
| `session_logs.py` | Confirm action names match convention |
| `attendance_buckets.py` | Confirm action names match convention |
| `class_sessions.py` | Confirm action names match convention |
| `holidays.py` | Confirm action names match convention |
| `deduction_masters.py` | Confirm action names match convention |
| `earning_masters.py` | Confirm action names match convention |
| `rooms.py` | Confirm action names match convention |
| `salary_templates.py` | Confirm action names match convention |
| `subject_hours.py` | Confirm action names match convention |
| `timetable_settings_templates.py` | Confirm action names match convention |

---

## Open Items

| Item | Decision needed |
|---|---|
| Which modules to wire up | Decided after module is built — covered separately |
| Payload display format | Key-value list assumed — confirm if a before/after diff format is preferred |
| Export (CSV / PDF) | Not in v1 — planned future addition |
| Audit log for audit log access | Should viewing the audit log itself be logged? |
