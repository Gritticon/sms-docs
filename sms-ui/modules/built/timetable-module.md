---
title: Timetables — Consolidated Documentation
type: module
project: sms-ui
module: Schedule Management
last-updated: 2026-03-18
---

# Timetables – Consolidated Documentation

Single reference for timetable functionality: overview, APIs, generate flow, fixed-slot constraints, template settings, database, and troubleshooting.

---

## 1. Overview

Timetables assign **periods** (subject, teacher, room) to each **class–section** per week. The system:

- Uses **timetable settings templates** for structure (periods per day, timings, working days, full/half day, breaks).
- **Generates** conflict-free draft timetables from subject allocation and optional fixed-slot rules.
- Supports **preview**, **publish**, **export** (Excel from API; PDF from Flutter), and **teacher timetable** view.
- Supports **delete** of timetables (backend returns 204; confirmation dialog in preview).

### Main features

| Feature | Description |
|--------|--------------|
| **Timetable settings templates** | Name, full-day/half-day periods and duration, working days, breaks (with “after which period”), room mandatory, status. |
| **Timetable list** | One timetable per section; filter by class/section; view status (draft/published). List data can be loaded in one call via `/timetables/list-data`. |
| **Generate timetable** | Class + section + template; set classes per subject; optional fixed-slot constraints and attendance periods; CSP-based generation. |
| **Edit & regenerate** | Open generate screen with stored options pre-filled; when opened from preview, template is pre-selected. Change and regenerate. |
| **Preview** | View timetable with slots, template info, class/section names; conflict summary for drafts. Cross-conflict banner with teacher/slot details (View details dialog), Ignore for now, and inline View details link. Layout: class/section card, status & visibility card, actions card (Activate, Regenerate, Revert, Delete). Staff name is hidden for subjects that have options. |
| **Publish** | Publish draft only when conflict-free (including no cross-section teacher double-booking). |
| **Regenerate** | One-click regenerate from preview using stored options and current teacher allotment/template. |
| **Revert to published** | Discard draft and restore slots from last published snapshot (when draft and snapshot exist). |
| **Delete** | Delete timetable and all slots; confirmation dialog in preview. List refreshes when returning after delete or status change. |
| **Visibility** | Admin can hide timetable from students (`is_visible_to_students`); toggle in preview. |
| **Export** | Download as **Excel (xlsx)** from API (`GET /timetables/{id}/export?format=xlsx`) or **PDF** from Flutter (client-side via `TimetablePdfExportService`). |
| **Teacher timetable** | Read-only view of all slots for a teacher. |
| **Cross-conflicts** | Same teacher, same day/period in multiple sections is detected; publish blocked until resolved. |

---

## 2. User flows

### Manage templates

- **List:** Timetable → Timetable templates (or nav to `timetable-templates`).
- **Add:** Add template → set name, full/half day periods and breaks (“after period”), working days, options → Save.
- **Edit:** Open template → edit → Save. No hard delete; use status Active/Inactive.
- **Delete:** Soft delete only (sets status INACTIVE); API returns 204 No Content.

### Create a timetable

- **List:** Timetable (main) shows sections with their timetables (and class names). Use “list-data” for a single load of classes, sections, and timetables.
- **Generate:** “Generate” or “Edit & regenerate” → Class, Section, Template → (optional) classes per subject, fixed-slot constraints, attendance periods → Generate.
- **Preview:** From list, open a timetable → preview (with conflicts if draft). Use **Regenerate** to re-run with current data; **Revert to published** to discard draft; **Delete** to remove timetable (with confirmation); toggle **Visible to students** as needed. When returning from preview after delete or status change, the list refreshes automatically.
- **Publish:** From preview, Publish (only if no conflicts and no cross-section teacher double-booking). Cross-conflict banner shown when same teacher is in same period in multiple sections.
- **Export:** Export as xlsx (API) or PDF (Flutter) from preview/list.

### Teacher timetable

- From staff/teacher context (e.g. staff detail or profile), open “Timetable” tab → read-only slots for that teacher.

---

## 3. API summary

All relevant list/detail responses are wrapped as `{ "success": true, "data": ..., "message": "..." }` unless noted.

### Timetables

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/timetables/` | List timetables (optional `class_id`, `section_id`, `visible_to_students_only`) |
| GET | `/timetables/bulk` | All timetables (optional `visible_to_students_only`) |
| GET | `/timetables/list-data` | Classes, sections, and timetables in one call (optional `visible_to_students_only`) |
| GET | `/timetables/cross-conflicts` | Cross-timetable teacher conflicts (optional `class_id`) |
| GET | `/timetables/{id}` | Timetable by ID with slots (optional `include_conflicts`; includes `is_visible_to_students`, `can_revert`) |
| GET | `/timetables/{id}/preview` | Full preview payload (timetable, slots, template, class, section, subject/teacher names, options) |
| GET | `/timetables/{id}/conflicts` | Re-run validation and return conflicts |
| GET | `/timetables/{id}/export?format=xlsx` | Export as Excel (only `xlsx` supported) |
| GET | `/timetables/teacher/{staff_id}` | All slots for a teacher |
| POST | `/timetables/generate/` | Generate (or regenerate) draft; create/update timetable and slots |
| POST | `/timetables/` | Create empty draft (no slots) |
| POST | `/timetables/{id}/regenerate` | Regenerate using stored options and current teacher/template data |
| POST | `/timetables/{id}/revert-to-published` | Discard draft and restore from last published snapshot |
| POST | `/timetables/{id}/publish` | Publish draft (only if conflict-free and no cross-conflicts); saves slot snapshot for revert |
| PATCH | `/timetables/{id}/visibility` | Set `is_visible_to_students` (body: `{ "is_visible_to_students": true/false }`) |
| DELETE | `/timetables/{id}` | Delete timetable and all slots; returns 204 No Content |

### Timetable settings templates

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/timetable-settings-templates/` | List templates (optional `active_only`) |
| GET | `/timetable-settings-templates/minimal` | Minimal list (id, name, status) for dropdowns |
| GET | `/timetable-settings-templates/{id}` | Template by ID |
| POST | `/timetable-settings-templates/` | Create template |
| PUT | `/timetable-settings-templates/{id}` | Update template |
| DELETE | `/timetable-settings-templates/{id}` | Soft delete (sets status INACTIVE); returns 204 No Content |

### Subjects for generate screen

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/classes/{class_id}/subjects/timetable-generate?section_id=X` | Minimal list (subject id, name) for timetable generate; used for single load. |
| GET | `/classes/{class_id}/subjects?section_id=X&group_by_subject=true` | Fallback when minimal endpoint unavailable. |

---

## 4. POST /timetables/generate/ – Contract

- **Full path:** `POST /timetables/generate/` (prefix as per app; e.g. `/timetables/generate/`).
- **Headers:** `Authorization: Bearer <token>`, `Content-Type: application/json`
- **Body:** JSON matching `TimetableGenerateRequest` (three required int IDs > 0: `class_id`, `section_id`, `timetable_settings_template_id`; rest optional).

### Request body

| Field | Type | Required | Notes |
|-------|------|----------|--------|
| `class_id` | int | Yes | `gt=0` |
| `section_id` | int | Yes | `gt=0` |
| `timetable_settings_template_id` | int | Yes | `gt=0` |
| `subject_periods` | Dict[int, int] | No | Subject ID → periods per week. Omit when user has not entered class counts; backend then uses subject hours. |
| `max_periods_per_teacher_day` | int | No | Optional. |
| `max_periods_per_teacher_week` | int | No | Optional. |
| `optional_groups` | Dict[int, List[int]] | No | UI sends `null` / omits. |
| `fixed_slot_constraints` | List[FixedSlotConstraintItem] | No | See §6. |
| `attendance_period_numbers` | List[int] | No | 1-based period numbers for attendance (e.g. [1, 2]). Stored on timetable; not used by CSP. |

### Response

- **200:** `TimetableGenerateResponse`: `data` contains `timetable`, `slots`, `conflict_summary` (same shape as `TimetableWithSlotsData`).
- **401:** Unauthorized. **422:** Validation error. **500:** Server/DB error.

Generation can take many seconds; Flutter uses `receiveTimeoutSeconds: 180` for the generate request.

---

## 5. Timetable generate flow (backend & UI)

- **One timetable per section:** Generating again for the same section updates the existing timetable and regenerates slots.
- **Stored options:** `subject_periods`, `fixed_slot_constraints`, `attendance_period_numbers` are stored on the timetable and returned in GET so the generate form can be pre-filled for "Edit & regenerate". When opening Edit from preview, the template is pre-selected from the timetable's `timetable_settings_template_id`.
- **Behaviour:** When the user has entered class counts, the frontend sends **subject_periods**. When none are entered, **subject_periods** is omitted and the backend uses subject-hour periods (by class/section). Generation is **subject-based**; optional subjects are considered as one variable per subject (not per option) so total periods match available slots.

### Generate screen (Flutter) – load and APIs

- **Load:** `initState` only starts `_loadClasses()`. When classes finish, sections load for the selected class. When sections finish, form data (templates if needed, subjects) loads once. So: one classes request, one sections request, one subjects request (and one templates request if needed). Subjects come from `GET /classes/{class_id}/subjects/timetable-generate?section_id=X` with fallback to `getSubjectsByClassGrouped(..., groupBySubject: true)`.
- **Validation before generate:** Class and section must be selected; template must be selected; total classes ≤ total slots per week; for each subject in fixed-slot constraints, total fixed slots (across all rows) ≤ that subject’s “Classes” value. Errors are shown in a **banner below the AppBar** (not AppMessage).
- **Key files:** `sms/lib/views/timetable_generate_view.dart`, `sms/lib/repositories/class_subject_repository.dart` (`getSubjectsForTimetableGenerate`), `sms/lib/repositories/timetable_repository.dart` (`generate`).

---

## 6. Fixed-slot constraints

**Purpose:** Pin a subject to selected days and periods (e.g. Drawing on Mon, Wed, Fri last period; Sports on Thu and Sat last period).

### UI

- Generate screen → “Fixed slot constraints” card. Per row: subject dropdown, day chips (Mon–Sun), period chips (Period 1…N, Last period). Add/remove rows. Theme colours used for chip labels and “Add constraint”.
- **Rule:** Total fixed slots for a subject **across all rows** ≤ that subject’s “Classes” value. The UI blocks adding a day/period (or new row) when the new total would exceed that. Inline message when total exceeds class count.

### API shape

Each item in `fixed_slot_constraints`:

| Field | Type | Description |
|-------|------|-------------|
| `subject_id` | int | Subject to place in each fixed slot. |
| `day_of_weeks` | List[int] | 1=Monday … 7=Sunday. Multiple allowed. |
| `period_specs` | List[int] | 1..N = period 1..N; **0** = last period of the day. Multiple allowed. |

**Semantics:** Subject must be placed in **every** slot in the Cartesian product of selected days × selected periods. Backend expands constraints (resolving “last period” per day from template), pre-fills those slots, then runs the CSP for the remaining periods.

**Conflict handling:** If fixed constraints conflict (duplicate slot requested twice, or subject has more fixed slots than period allowance), generation proceeds with best-effort placement (first wins for duplicates; cap at max periods for exceed). The conflicts are collected and returned in `conflict_summary` (type `fixed_constraint_duplicate` or `fixed_constraint_exceed_periods`) so the UI can display them.

---

## 7. Timetable settings template

- **Full day:** `full_day_periods_per_day`, `full_day_period_duration_minutes`, `full_day_breaks` (name, duration_minutes, period_after). Used for working days marked as full.
- **Half day (optional):** `half_day_periods_per_day`, `half_day_period_duration_minutes`, `half_day_breaks`. Used for working days marked as half; if not set, generation falls back to full-day values.
- **Working days:** JSON with `day` (1–7), `type` ("full" or "half"), optional `periods` for half days.
- **Break placement:** `period_after` = break occurs after period N; period N+1 is non-instructional (excluded from eligible slots).
- **Options:** room_mandatory, status (active/inactive), attendance_period_numbers (optional). No “effective from” or academic year on template (removed).
- **Delete:** Soft delete only (set status INACTIVE); API returns 204. No hard delete in UI or API.

---

## 8. Database tables (timetable-related)

Per-school dynamic tables (pattern `{school_id}_ModelName`):

| Table | Purpose |
|-------|---------|
| `{school_id}_Timetable` | One row per section: class_id, section_id, timetable_settings_template_id, status, version, attendance_period_numbers, subject_periods, fixed_slot_constraints, published_at, **published_slots_snapshot** (JSON, slot list at last publish for revert), **is_visible_to_students** (bool, default true), created_at, updated_at. **No** academic_year_id, effective_from, effective_to. |
| `{school_id}_TimetableSlot` | One row per period: timetable_id, day_of_week, period_number, subject_id, teacher_id, room_id, created_at, updated_at. |
| `{school_id}_TimetableSettingsTemplate` | Template definition (full/half day, working_days, breaks, etc.). **No** effective_from. |

Referenced by timetables: `{school_id}_Class`, `{school_id}_Section`, `{school_id}_Subject`, `{school_id}_Staff`, `{school_id}_Room`, `{school_id}_ClassSubject`, `{school_id}_SubjectHour`.

---

## 9. Legacy column migration (existing deployments)

If your database still has old columns, drop them so the table matches the current model:

- **Timetable:** Drop `academic_year_id`, `effective_from`, `effective_to` from `{school_id}_Timetable`.
- **Timetable settings template:** Drop `effective_from` from `{school_id}_TimetableSettingsTemplate`.

**Option 1 – Manual DDL (replace `{school_id}` with actual ID, e.g. 220599):**

```sql
ALTER TABLE `{school_id}_Timetable` DROP COLUMN IF EXISTS academic_year_id;
ALTER TABLE `{school_id}_Timetable` DROP COLUMN IF EXISTS effective_from;
ALTER TABLE `{school_id}_Timetable` DROP COLUMN IF EXISTS effective_to;
ALTER TABLE `{school_id}_TimetableSettingsTemplate` DROP COLUMN IF EXISTS effective_from;
```

**Option 2 – Script:** From repo root, with API environment set: `cd sms-api && python scripts/drop_timetable_legacy_columns.py`. The script discovers `*_Timetable` and `*_TimetableSettingsTemplate` tables and drops the columns only if they exist. Note: `get_dynamic_model` in the app only **adds** missing columns; it does **not** drop columns, so this migration is required for existing DBs that have these columns.

---

## 10. Generation algorithm (summary)

See **TIMETABLE_ALGORITHM.md** for a full description with use cases and how staff usage is maximised.

- **Inputs:** Class, section, template, subject_periods (or subject hours from backend), optional_groups (optional), fixed_slot_constraints, teacher assignments from teacher allotment.
- **Template:** Working days, periods per day (full/half), eligible slots; breaks exclude periods after `period_after` but do not remove slots from the slot list used for placement.
- **Fixed slots:** Expanded and pre-filled; remaining periods placed by CSP.
- **Hard constraints:** One slot per (day, period); one teacher per (day, period); max periods per day per subject (default 3). **Cross-section teacher capacity:** A teacher cannot be in more than one section at the same (day, period). Both initial generation and regeneration use an occupied-teacher-slot map from sibling sections (draft + published timetables) and reject candidates that would double-book.
- **Heuristics:** Spread subjects across days; avoid consecutive same subject where possible. **Placement order:** Mandatory subjects first, then by periods descending (high-load subjects first), then optional group. **Shuffle + retry:** Up to 5 attempts with randomly shuffled placement units to find a feasible solution when one exists (e.g. rotated subject order across sections). **Teacher gap-filling:** When ordering candidate slots, prefer slots that fill gaps in a teacher's week (same day, adjacent period already occupied) to maximise staff usage and compact schedules.
- **Not considered:** Holidays, room conflicts; teacher day/week caps only in teacher-based path. Optional subjects are treated as **one variable per subject** (not per option) when using subject_periods from the UI.

---

## 11. Troubleshooting (generate)

- **Timeout:** Use 180s receive timeout for `POST /timetables/generate/` in Flutter; ensure proxy/ALB does not time out earlier.
- **422:** Check request body: class_id, section_id, timetable_settings_template_id must be int > 0; fixed_slot_constraints if present must have subject_id, non-empty day_of_weeks and period_specs.
- **500 “Column 'academic_year_id' cannot be null”:** The DB table has that column as NOT NULL but the app no longer sends it. Run the legacy column migration (§9) to drop academic_year_id, effective_from, effective_to from Timetable and effective_from from TimetableSettingsTemplate.
- **422 “Could not place … Teacher has no free period”:** The generator retries up to 5 times with different subject placement orders. If all attempts fail, the teacher truly has no free (day, period) across sibling sections. Add or redistribute teachers for that subject, reduce subject periods, or adjust template/working days.
- **422 “Could not place all subjects without conflict”:** Total periods requested may exceed available slots. Ensure subject_periods sends one entry per subject (not per option) and total periods ≤ total slots after fixed slots. Check max 3 periods per day per subject.
- **Publish 400 “Teacher double-booked”:** Same teacher is assigned the same (day, period) in more than one section. The generator now avoids this during both generation and regeneration by respecting sibling sections' occupied slots. If conflicts persist (e.g. legacy data, manual edits), use Teacher Allotment to change assignments or regenerate affected timetables.
- **Revert 400 “No published version to revert to”:** Only drafts can be reverted, and only when a snapshot exists (i.e. the timetable was published at least once). Publish first to create a snapshot.

---

## 12. Related modules

- **Teacher allotment** – Links teachers to class–section–subject; used when generating slots.
- **Subject hours** – Periods per week per class/section/subject; used when subject_periods is not sent.
- **Rooms** – Used in slots; managed elsewhere.
- **Holidays** – Non-working days (not used in slot eligibility in current generation).

---

## 13. Key files

| Layer | Path |
|-------|------|
| Flutter generate view | `sms/lib/views/timetable_generate_view.dart` |
| Flutter preview view | `sms/lib/views/timetable_preview_view.dart` |
| Flutter timetable list/view | `sms/lib/views/timetable_view.dart` |
| Flutter template edit | `sms/lib/views/timetable_template_edit_view.dart` |
| Flutter PDF export | `sms/lib/services/timetable_pdf_export_service.dart` |
| Flutter repos | `sms/lib/repositories/timetable_repository.dart`, `sms/lib/repositories/class_subject_repository.dart` |
| Backend routes | `sms-api/app/api/routes/timetables.py`, `sms-api/app/api/routes/classes.py`, `sms-api/app/api/routes/timetable_settings_templates.py` |
| Backend services | `sms-api/app/services/timetable_generation_service.py`, `sms-api/app/services/timetable_service.py`, `sms-api/app/services/timetable_export_service.py`, `sms-api/app/services/timetable_settings_template_service.py`, `sms-api/app/services/class_subject_service.py` |
| Backend models/schemas | `sms-api/app/models/timetable.py`, `sms-api/app/models/timetable_slot.py`, `sms-api/app/models/timetable_settings_template.py`, `sms-api/app/schemas/timetable.py`, `sms-api/app/schemas/timetable_slot.py`, `sms-api/app/schemas/timetable_settings_template.py` |
