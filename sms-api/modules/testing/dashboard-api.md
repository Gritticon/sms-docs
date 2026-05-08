---
title: "Dashboard Widget Persistence API"
type: planning-stub
project: sms-api
module: Dashboard
last-updated: 2026-03-28
---

> Auto-promoted to testing on 2026-03-28 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — The SMS-UI Dashboard spec defines a 21-widget customisable grid (revised 2026-03-26). The API needed to save and load each staff member's widget layout exists as stubs. Widget data endpoints are spread across existing module routes but require role-scoped filtering.

---

## Layout Persistence Endpoints

These two endpoints are the core of the dashboard API. They are implemented as stubs in `sms-api` but need full spec coverage.

### `GET /staff/me/dashboard`
Returns the authenticated staff member's saved widget layout.

**Response:**
```json
{
  "success": true,
  "data": {
    "dashboard_layout": [
      { "widget_id": "quick_links", "position": 0, "config": { "module_slugs": ["student-management", "exams"] } },
      { "widget_id": "student_attendance", "position": 1 },
      { "widget_id": "staff_presence", "position": 2 }
    ]
  }
}
```

- If no layout has been saved: return `"dashboard_layout": null` — the frontend generates and saves the default layout on first render
- Layout is per-user per-school (tied to the staff member's active school context)

### `PUT /staff/me/dashboard`
Overwrites the authenticated staff member's entire widget layout.

**Request body:**
```json
{
  "dashboard_layout": [
    { "widget_id": "quick_links", "position": 0, "config": { "module_slugs": ["student-management", "exams"] } },
    { "widget_id": "student_attendance", "position": 1 }
  ]
}
```

- Backend does not validate widget IDs against permissions — that is a frontend concern
- Backend does not validate `config` contents

---

## Widget Data Endpoints

Each widget fetches its own data from existing module endpoints. The critical requirement is **role-scoped filtering** — the backend must filter responses based on the authenticated user's role and class/department assignments, not just their module permissions.

### Role-scoping requirement

Several widgets show different data depending on who is looking. The backend derives the role context from the authenticated session — the frontend sends no additional filter parameters.

| Widget ID | Role-scoping behaviour |
|---|---|
| `student_attendance` | Class Teacher → filtered to their assigned classes only; Principal/VP → school-wide |
| `open_issues` | Class Teacher → complaints involving their class only; Principal → all complaints |
| `pending_approvals` | HR role → leave requests only; Accounts role → fee waivers only; Principal → all |
| `inbox` | Each user → their own messages and role-relevant announcements only |
| `student_welfare` | Class Teacher → their class only; Counsellor and Principal → school-wide |
| `pending_results` | Subject Teacher → their subject exams only; Exam Controller → all exams |
| `my_pending_tasks` | Always scoped to the logged-in user regardless of role |

### Widget-to-endpoint mapping

| Widget ID | Data endpoint | Route file | Status |
|---|---|---|---|
| `student_attendance` | `GET /students/attendance?date=today` + `GET /students/` (count) | `attendance_buckets.py`, `students.py` | ✓ Routes exist — add role-scope filter |
| `staff_presence` | `GET /staff/` + `GET /staff/attendance?date=today` | `staff.py`, `staff_attendance.py` | ✓ Routes exist — add role-scope filter |
| `substitution_coverage` | `GET /substitute/?date=today` | `substitute.py` | ✓ Route exists |
| `live_sessions` | `GET /class-sessions/?status=active` | `class_sessions.py` | ✓ Route exists |
| `classes_coverage` | `GET /classes/` + `GET /sections/` + `GET /timetables/?date=today` | `classes.py`, `sections.py`, `timetables.py` | ✓ Routes exist |
| `exam_schedule` | `GET /exams/?upcoming=true` | `exams.py` | ✓ Route exists |
| `pending_results` | `GET /exams/results?status=pending_entry` | `exams.py` | Needs new query param — add role-scope filter |
| `open_issues` | `GET /support-tickets/?status=open` | `support_tickets.py` | ✓ Route exists — add role-scope filter |
| `pending_approvals` | `GET /staff/leave-requests?status=pending` + fee/enrolment pending endpoints | TBD | Blocked — leave request and enrolment approval endpoints not yet built |
| `fee_collection` | `GET /fees/collection?month=current` | TBD | Blocked — fee module not built |
| `late_arrivals` | `GET /students/attendance?type=late&date=today` | `attendance_buckets.py` | Needs new query param `type=late` |
| `upcoming_events` | `GET /holidays/?upcoming=true` + `GET /events/?upcoming=true` | `holidays.py` | Events endpoint TBD — holidays route exists |
| `inbox` | `GET /messages/unread` + `GET /announcements/unread` | TBD | Blocked — communication hub not built |
| `transport_status` | `GET /transport/routes?status=active` | TBD | Blocked — transport module not built |
| `student_welfare` | `GET /students/attendance?consecutive_absent_min=3` + `GET /incidents/?status=open` | `attendance_buckets.py` (partial) | Consecutive absence tracking needs new backend logic; incidents endpoint TBD |
| `library_status` | `GET /library/books/overdue` + `GET /library/issues?date=today` | TBD | Blocked — library module not built |
| `quick_links` | No data fetch — static navigation | — | ✓ No endpoint needed |
| `my_day` | `GET /timetables/?staff_id=me&date=today` | `timetables.py` | ✓ Route exists |
| `my_classes_health` | `GET /class-sessions/?staff_id=me` + `GET /students/attendance?class_id=` | `class_sessions.py`, `attendance_buckets.py` | ✓ Routes exist — compose on frontend |
| `my_attendance` | `GET /staff-attendance/?staff_id=me` | `staff_attendance.py` | ✓ Route exists |
| `my_pending_tasks` | Aggregates from `pending_results` + `pending_approvals` + `inbox` scoped to user | TBD | Needs unified aggregation endpoint or frontend composition |

---

## Key Decisions to Make Before Writing Spec

1. **Role-scoping implementation** — Does the backend add role-filtering middleware to existing endpoints, or are there separate dashboard-specific endpoints (e.g. `/dashboard/student-attendance`) that wrap the module endpoints with role context? Middleware is simpler but may affect non-dashboard consumers. Separate endpoints are explicit but add routes.

2. **`pending_approvals` aggregation** — Leave approvals (Staff Management), fee waivers (Fee module), enrolment approvals (Student Management) live in different modules. Does the backend expose a unified `/approvals/pending` endpoint that aggregates all pending items by role, or does the frontend call each module separately?

3. **`my_pending_tasks` aggregation** — Same question as above — unified endpoint vs frontend composition from multiple sources.

4. **Consecutive absence tracking for `student_welfare`** — Requires computing attendance streaks per student. Should this be a computed field returned by the attendance endpoint (e.g. `?consecutive_absent_min=3`) or a separate analytics endpoint?

5. **Caching** — Widget data should be cached (seconds to minutes) to avoid hammering module endpoints on every dashboard load. Where: Redis on ECS? Cache invalidation strategy per widget type?

6. **`DELETE /staff/me/dashboard`** — Should a delete/reset endpoint exist, or is reset handled by the frontend sending the default layout via PUT?

---

## Related Docs

- SMS-UI Dashboard spec: `/doc/sms-ui/modules/testing/dashboard-module`
- Module registry: `/doc/sms-ui/architecture/modules-and-submodules`
- Permission implementation: `/doc/architecture/permission-implementation`
- System architecture: `/doc/architecture/system-architecture`
