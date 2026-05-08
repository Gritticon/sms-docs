---
title: Staff Attendance — Redesign Plan
type: planned
status: planned
project: platform
module: Staff Attendance
last-updated: 2026-03-29
---

# Staff Attendance — Redesign Plan
### Cursor Implementation Instructions | No code, step-by-step

## Context

The staff attendance system is being redesigned from a **manual-first marking form** into a **multi-source, pipeline-based, exception-driven system**.

- Geofence is the **primary source** — system marks attendance automatically
- Manual is a **fallback/correction** — not the primary daily action
- Future sources (biometric, face scan) plug in without architectural changes
- Admin's job = **review exceptions**, not mark attendance daily

Full product context: `docs/product/staff-attendance-location-feature.md`

> **Before implementing:** Read `docs/product/product-philosophy.md`. Every screen, flow, and decision in this plan must pass the design test defined there. Key checks for this module: manual override one tap away at all times, system suggests corrections never forces them, audit trail shows what the system did vs what the admin approved.

---

## Part 1 — Database Schema

### Step 1.1 — Create `attendance_events` table (new)

Create a new dynamic table `{school_id}_attendance_events` with these columns:

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK, auto-increment | |
| `staff_id` | int, FK to staff table | |
| `event_type` | enum: `clock_in`, `clock_out`, `status_override` | |
| `source` | enum: `geofence`, `manual`, `biometric`, `face_scan` | |
| `event_time` | datetime | actual time of event |
| `date` | date | derived from event_time, indexed |
| `status` | enum: `present`, `absent`, `late`, `half_day`, `holiday` | only used when event_type = `status_override` |
| `reason` | varchar, nullable | for manual overrides |
| `notes` | text, nullable | |
| `created_by` | int, FK to staff (admin who created manual entry) | nullable; null = system-generated |
| `created_at` | datetime | |

Add composite index on `(staff_id, date)` and `(date, source)`.

---

### Step 1.2 — Update `{school_id}_staff_attendance` table (existing)

This table becomes the **resolved/final record** table. Add these columns:

| Change | Column | Details |
|---|---|---|
| Add | `resolved_source` | enum: `geofence`, `manual`, `biometric`, `face_scan` — which source produced the final record |
| Add | `is_exception` | boolean, default false — flagged by processing layer when anomaly detected |
| Add | `exception_reason` | varchar, nullable — e.g. "No geofence event found", "Conflicting sources" |
| Add | `reviewed_by` | int, nullable FK — admin who reviewed/cleared the exception |
| Add | `reviewed_at` | datetime, nullable | |
| Add | `is_finalized` | boolean, default false — set to true when day-boundary processing completes for this date |
| Update | `source` enum | Deprecated — keep for historical records only, never written to by new code |
| Add | `resolved_source` | enum: `geofence`, `manual`, `biometric`, `face_scan` — **authoritative source field going forward**, replaces `source` |

No columns removed. `source` is kept for historical records but `resolved_source` is the single authoritative field for all new records.

---

### Step 1.3 — Update school settings table

Add geofence config columns to the school settings table:

| Column | Type | Notes |
|---|---|---|
| `geofence_latitude` | decimal(10,8), nullable | |
| `geofence_longitude` | decimal(11,8), nullable | |
| `geofence_radius_meters` | int, default 100 | |
| `geofence_enabled` | boolean, default false | |

---

### Step 1.4 — Create `{school_id}_staff_shift_templates` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `name` | varchar | e.g. "Morning Shift" |
| `shift_start` | time | |
| `shift_end` | time | |
| `grace_minutes_start` | int, default 0 | late detection threshold |
| `grace_minutes_end` | int, default 0 | early exit detection threshold |
| `working_days` | json | array of day names e.g. `["MON","TUE","WED","THU","FRI"]` |
| `break_windows` | json | array of `{name, start, end}` objects — exits during these ignored by geofence |
| `is_default` | boolean, default false | one default per school |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

### Step 1.5 — Create `{school_id}_staff_shift_assignments` table (new)

| Column | Type | Notes |
|---|---|---|
| `id` | int, PK | |
| `staff_id` | int, FK | |
| `shift_template_id` | int, FK | |
| `effective_from` | date | |
| `effective_until` | date, nullable | null = currently active |
| `created_at` | datetime | |

Add unique constraint on `(staff_id, effective_from)`.

---

## Part 2 — Backend API Pipeline

### Step 2.1 — Create `POST /attendance/event` endpoint (new)

Single unified endpoint that accepts attendance events from any source. **Replaces `POST /staff-attendance/bulk` entirely — delete the old endpoint.**

Request body: `staff_id`, `event_type`, `source`, `event_time`, `status` (optional, for overrides), `reason` (optional), `notes` (optional).

Rules:
- **Reject any `event_time` with a date in the future** — return 400 with message "Future date attendance is not allowed"
- **Reject any `event_time` with a date before today** only if it would create a record on a date that already has a finalized transaction — return 400 with message "This date is already finalized"
- Validate `staff_id` belongs to the school (from auth token)
- If `source = geofence | biometric | face_scan`: `created_by` = null (system event)
- If `source = manual`: `created_by` = requesting user's staff_id
- Save to `attendance_events` table
- After save, trigger the day-boundary check (see Step 2.2) then the processing function for that `(staff_id, date)` pair

---

### Step 2.2 — Day-boundary check + attendance processing function (internal, not an endpoint)

**No background job scheduler is used.** Day finalization is triggered naturally by incoming events.

#### Day-boundary check

Run this on **both** of these triggers — no scheduler required:

- Every time a new event arrives via `POST /attendance/event`
- Every time the attendance screen is opened via `GET /staff-attendance/init` or `GET /attendance/dashboard`

This ensures finalization happens even on days with no incoming events (e.g. holidays, weekends) — the next person who opens the attendance screen triggers it.

Logic:

1. Get `request_date` — from the event's `event_time` (POST) or from today's date (GET)
2. Get `latest_staged_date` — the most recent date in `staff_attendance` where `is_finalized = false`, scoped to the current `school_id`
3. If `request_date > latest_staged_date`:
   - **Loop** — multiple days may have passed (weekends, holidays, no-event days):
     - While `latest_staged_date < request_date`:
       - Run the processing function for **all staff** on `latest_staged_date`
       - Set `is_finalized = true` on all `staff_attendance` records for that date
       - Advance `latest_staged_date` by one day
   - For GET requests: proceed to return data for `request_date` as usual
   - For POST requests: the new event becomes the first staged record for `request_date`
4. If `request_date == latest_staged_date`: no boundary crossed, proceed normally

This handles weekends, holidays, and any multi-day gap — every skipped date gets finalized in order before proceeding.

#### Processing function

This function takes `(school_id, staff_id, date)` and:

1. Fetches all events for that `(staff_id, date)` ordered by `event_time`
2. Fetches the staff member's active shift template for that date — query `staff_shift_assignments` where `staff_id = X` and `effective_from <= date` and (`effective_until >= date` or `effective_until = null`). If no assignment found → fall back to the school's default shift template (`is_default = true`). If no default exists → set `is_exception = true`, `exception_reason = 'No shift template configured'`, mark as `absent` at finalization
3. Applies resolution logic:
   - If a `status_override` event exists → use it as final status (manual always wins; if multiple overrides exist, use the one with the latest `created_at`)
   - Else if `clock_in` event exists → derive status from event_time vs shift_start + grace_minutes
   - Else → at finalization, mark as **`absent`** (not `not_marked`) with `is_exception = true`, `exception_reason = "No attendance event received"`
   - `not_marked` is an intra-day state only — valid while the day is still open. It must never appear in a finalized record.
4. Detect anomalies and set `is_exception = true` with a reason when:
   - Clock-in time is after grace period (late)
   - No clock-out event found
   - Conflicting events from different sources on same day
   - Manual `status_override` on top of a `geofence` record (flag for awareness — audit log retains full trail)
5. Write/update the final record in `staff_attendance` table
6. Set `resolved_source` to the source that produced the final status

---

### Step 2.3 — Create `GET /attendance/dashboard` endpoint (new)

Returns today's attendance summary for the school. Response must include:
- `total_staff` count
- `present` count
- `absent` count
- `late` count
- `not_marked` count (staff with no events today — intra-day only)
- `exceptions` count — **total unreviewed exceptions across last 7 days** (`is_exception = true` and `reviewed_by = null`)
- `oldest_exception_days_ago` — how many days ago the oldest unreviewed exception occurred (drives the "oldest from X days ago" label in the UI)
- `by_department` breakdown of today's counts

---

### Step 2.4 — Update `GET /staff-attendance/init` endpoint (existing)

Add to the response:
- Per staff record: `is_exception`, `exception_reason`, `resolved_source`, `reviewed_by`
- Top-level: `exceptions_count`
- New filter parameter `?filter=exceptions` — returns only unreviewed exception records

---

### Step 2.5 — Create `POST /attendance/review/{attendance_id}` endpoint (new)

Marks an exception as reviewed by the current admin.

Sets `reviewed_by` = requesting admin's staff_id, `reviewed_at` = now, `is_exception` = false.

### Step 2.5b — Create `DELETE /attendance/review/{attendance_id}` endpoint (new)

Undoes a review action — used by the undo toast in the UI (4-second window).

Resets `reviewed_by` = null, `reviewed_at` = null, `is_exception` = true on the attendance record.

Only the admin who performed the review can undo it, and only if `reviewed_at` is within the last 60 seconds (server-side guard to prevent abuse beyond the toast window).

---

### Step 2.6 — Create `GET /attendance/event-log` endpoint (new)

Returns all raw `attendance_events` for a `(staff_id, date)`.

Query params: `staff_id`, `date`.

Used by admin when drilling into a specific staff record to see the full event history.

---

## Part 3 — Flutter UX Redesign

> Key UX principles: exception-driven primary action, source-agnostic display, single primary CTA per screen, progressive disclosure, spatial continuity between screens.

---

### Step 3.1 — Dashboard Widget (new component)

Location: embedded in the main SMS dashboard screen.

Design as a summary card:
- Title: "Staff Attendance — Today"
- Horizontal row of stat chips: Present (green), Absent (red), Late (amber), Not Marked (grey)
- Highlighted `Exceptions` count in orange — this is the attention driver
- Single CTA button: "Review Exceptions" — navigates to Attendance Review screen with exceptions filter pre-applied
- Data source: `GET /attendance/dashboard`

States:
- If `exceptions = 0`: show green checkmark state "All clear"
- If `not_marked > 0` after configurable time (e.g. 10am): show soft warning
- Touch target for CTA minimum 48×48dp

---

### Step 3.2 — Redesign `StaffAttendanceView` — Review + Edit Modes

Rename the screen conceptually from "Mark Attendance" to "Attendance Review".

**Review Mode (default on load):**
- Top: Date picker (existing, defaults to today)
- Top: Filter chips — All | Exceptions | Present | Absent | Late | Not Marked
- Default filter = **Exceptions** (not "All")
- **Exceptions filter spans the last 7 days** — not just today. Shows all unreviewed exceptions across recent dates. Display a date label on each row when viewing multi-day exceptions (e.g. "Yesterday", "2 days ago")
- A summary line above the list: "X unreviewed exceptions — oldest from Y days ago" — keeps admin aware of backlog
- Each staff row shows:
  - Staff name + department
  - Status chip (green/red/amber/grey)
  - Source badge (small, subtle): "Geo" | "Manual" | "Bio"
  - If `is_exception = true`: orange warning icon + `exception_reason` as subtitle
  - **Exception rows show a checkbox directly** — no mode switching needed for bulk exception correction
  - Tapping the row (not the checkbox) opens the Staff Attendance Detail drawer (Step 3.3)
- When one or more exception checkboxes are selected:
  - A bulk action bar slides up from the bottom
  - Bar shows: "Mark X selected as →" with status chips (Present, Absent, Late, Half Day)
  - Selecting a status calls `POST /attendance/event` with `source: manual`, `event_type: status_override` for each selected staff
  - Bar dismisses on completion or on deselect-all
- "Review" button on individual exception rows (when not in bulk selection):
  - Calls `POST /attendance/review/{id}`
  - Shows a brief undo toast: "Exception marked as reviewed — Undo" (auto-dismisses in 4 seconds)
  - Undo calls a `DELETE /attendance/review/{id}` to revert the reviewed state

**Edit Mode (full list correction, for non-exception staff):**
- Accessed via toolbar button "Edit Attendance" — for bulk marking staff who are not flagged as exceptions
- Full staff list with checkboxes visible
- Selection-based bulk marking (existing behavior)
- "Apply" calls `POST /attendance/event` with `source: manual`, `event_type: status_override`
- "Done" exits Edit Mode and returns to Review Mode

---

### Step 3.3 — Staff Attendance Detail Drawer (new)

Triggered by tapping a staff row in Review Mode. Renders as a bottom sheet or side drawer (not a full screen push).

Contents:
- Staff name, photo, department
- Today's final status (large, prominent)
- Source of record: "Marked by Geofence at 8:47am" or "Manually marked by [Admin Name]"
- **Event Log** — timeline of raw events for this staff on this date:
  - Each event: time, event type (clock in / clock out / override), source badge, created by or "System"
  - Data source: `GET /attendance/event-log?staff_id=X&date=Y`
- **"Correct Attendance" button** — behaviour depends on current resolved source:
  - If `resolved_source = geofence | biometric | face_scan`: show a confirmation dialog first — "This record was set by [Source]. Overriding it will be logged in the audit trail. Continue?" — with Cancel and Confirm actions
  - If `resolved_source = manual` or record is unresolved: open correction form directly (no confirmation needed)
  - On confirm, reveal inline correction form (progressive disclosure):
    - Status dropdown
    - Reason text field (required if absent)
    - Time override fields (optional)
    - Submit calls `POST /attendance/event` with `source: manual`, `event_type: status_override`
- Correction form hidden by default, shown only after confirmation (or directly for manual records)

---

### Step 3.4 — Update Attendance Summary Cards (existing widget)

Update `attendance_summary_cards.dart`:
- Add "Exceptions" card with count and orange color
- Add secondary source breakdown row: "Geofence: X | Manual: Y | Not Marked: Z"
- Keep existing Present / Absent / Late / Half Day cards

---

### Step 3.5 — Setup Section (new, isolated module)

**This is a separate section from all operational screens (Attendance Review, Leave Overview, Payroll Workspace) and from general School Settings.**

Location: Main navigation → "Setup" (its own nav entry).

Reason for isolation: operational users doing daily work should never accidentally land here. This section is for principals and school admins doing one-time or infrequent configuration.

Named "Setup" — not "Attendance Setup" or "Leave Setup" — so new modules can plug in without renaming or restructuring. Each new attendance source added to the `source` enum in `attendance_events` gets a corresponding config entry under Setup → Attendance → Sources. The structure mirrors the pipeline.

```
Setup
  ├── Attendance
  │   ├── Shift Templates
  │   └── Sources
  │       ├── Geofence              ← current
  │       ├── Biometric             ← future placeholder
  │       └── Face Scan             ← future placeholder
  └── Leave
      ├── Leave Types               ← owned by leave management plan
      └── Early Permission Config   ← owned by leave management plan
```

Requires elevated permission throughout — not visible to regular attendance admins.

---

#### Step 3.5a — Geofence Configuration

Location: Setup → Attendance → Sources → Geofence

Contents:
- Map widget with draggable pin + visible radius circle
- Radius slider (50m – 500m)
- Enable/disable geofence toggle
- Save button
- Reads/writes: `geofence_latitude`, `geofence_longitude`, `geofence_radius_meters`, `geofence_enabled`

---

#### Step 3.5b — Shift Templates

Location: Setup → Attendance → Shift Templates

**Templates list:**
- List of shift templates (name, shift time, working days)
- Add / Edit / Deactivate template
- One template markable as "Default" — applied to staff with no explicit assignment

**Template detail — two tabs:**

*Details tab:*
- Template form fields: name, shift start, shift end, grace period (minutes), working days (multi-select chips), break windows (add/remove rows with name + start/end time)

*Assigned Staff tab:*
- List of all staff currently assigned to this template
- Department filter
- "Assign Staff" button → staff picker (individual or by department) → effective from date picker (defaults to today) → confirm
- Remove individual staff from this template
- "Bulk Reassign" — select staff → move to another template → effective from date picker → confirm
- On any assignment change: previous active assignment for that staff is auto-closed (`effective_until = new effective_from − 1 day`)

---

### Step 3.7 — Update `StaffAttendancePresenter` data flow

- Manual status change → call `POST /attendance/event` with `source: manual`, `event_type: status_override`
- **Delete all calls to `POST /staff-attendance/bulk`** — no migration path, the old endpoint is removed
- After a successful event post: re-fetch only the affected staff record and update state locally — do not trigger a full list reload on every correction

---

## Part 4 — Implementation Order

Run in sequence. Each step is independently testable before the next begins.

```
1.  Delete POST /staff-attendance/bulk endpoint and all callers
2.  DB migrations — add is_finalized, resolved_source, is_exception, reviewed_by, reviewed_at to staff_attendance; create attendance_events, shift_templates, shift_assignments tables; add geofence columns to school settings (Steps 1.1 → 1.5)
3.  POST /attendance/event endpoint with future-date guard (Step 2.1)
4.  Day-boundary check with multi-day loop (Step 2.2)
5.  Attendance processing function — not_marked → absent at finalization, last created_at wins for multi-override (Step 2.2)
6.  GET /attendance/dashboard endpoint with 7-day exception window (Step 2.3)
7.  Update GET /staff-attendance/init with resolved_source, is_exception, multi-day exception filter (Step 2.4)
8.  POST /attendance/review + DELETE /attendance/review endpoints (Step 2.5 + 2.5b)
9.  GET /attendance/event-log endpoint (Step 2.6)
10. Dashboard widget in Flutter (Step 3.1)
11. Redesign StaffAttendanceView — Review Mode with inline exception checkboxes + bulk action bar + undo toast; Edit Mode for non-exception bulk marking (Step 3.2)
12. Staff Attendance Detail Drawer with confirmation dialog (Step 3.3)
13. Update Attendance Summary Cards (Step 3.4)
14. Setup section — Attendance: Geofence Config (Step 3.5a) + Shift Templates with Assigned Staff tab and effective_from assignment logic (Step 3.5b); Leave sub-section structure defined here, screens implemented in leave management plan
15. Update StaffAttendancePresenter data flow (Step 3.7)
```

---

## Out of Scope (Next Phase)

These are intentionally excluded until the above foundation is stable:

- Geofence background service in Flutter shell (SMS-UI Shell)
- Attendance Rules Engine (auto leave deduction, LOP)
- Biometric / Face Scan hardware integration

---

## Leave Management Extension

The leave management module extends the attendance enums:
- `on_leave` added to the status enum in `staff_attendance` and `attendance_events`
- `leave` added to the source enum in `attendance_events`

These enum changes are owned by the leave management plan and must be applied together with Step 1 of this implementation order.

See: `docs/product/planned/leave-management-plan.md`
