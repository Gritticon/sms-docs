---
title: Teacher Allotment — Feature Document
type: module
project: sms-ui
module: Schedule Management
last-updated: 2026-03-18
---

# Teacher Allotment – Feature Document

Overview, user flow, architecture, and review of the Teacher Allotment feature.

---

## 1. Overview

**Purpose:** Assign teachers to subjects for each class and section. One subject can have multiple teachers; each class–section has a set of subjects, each with zero or more assigned teachers. **Subjects with options** (e.g. “Second Language” with French, German, Spanish) can have **different teachers per option**; the UI shows one card per option and save/load is per (class, section, subject, subject_option).

**Module:** Timetable → **Teacher Allotment** (submodule ID: 606).

**Where it fits:** Teacher allotment data is used by **Timetable Generation** (subject hours and teacher assignments). If no subject hours or teacher allotments exist, timetable generation reports that the user should define subject hours or assign subjects via Teacher Allotment first.

---

## 2. User Flow

1. User opens **Teacher Allotment** from navigation (Timetable module).
2. **Class** dropdown: user selects a class. The app loads sections and per-section subjects (minimal) in one call.
3. **Section** and **Role** (optional): user can filter by section and by staff role (e.g. “Teacher”) to load staff for the teacher picker.
4. **View / Edit** mode toggle:
   - **View:** Read-only table: sections × subjects (or subject–option rows) with assigned teachers.
   - **Edit:** User can add/remove teachers per subject (or per option) per section (pending state in memory).
5. **Save** (per section): User finalises changes for a section. The app sends all allotments for that class–section to the API; backend replaces existing ClassSubject rows and returns the updated list. UI shows success and refreshes that section’s data.

**Data shape:** For each class–section, data is a list of “subject (+ optional option) + list of teachers”. For subjects **without** options: one entry per subject. For subjects **with** options: one entry per option (e.g. “Second Language – French”, “Second Language – German”). Saving sends `allotments`: list of `{ subject_id, subject_option_id?, teacher_ids }`; backend replaces all ClassSubject rows for that class–section (each row may have `subject_option_id` set for option-subjects).

---

## 3. Architecture

### 3.1 Frontend (Flutter – UI only)

| Layer | File | Responsibility |
|-------|------|----------------|
| View | `lib/views/teacher_allotment_view.dart` | Class/section/role selectors, View/Edit toggle, table (sections × subject/option cards × teachers), Save button, loading/error display. Pending edits keyed by `(subjectId, optionId)` in `_pendingBySection` (key format: `subjectId_optionId`, with `0` for no option). One card per option for option-subjects; card title e.g. “Subject – Option name”. |
| Presenter | `lib/presenters/teacher_allotment_presenter.dart` | Load subjects (by class–section or minimal for class), save allotments. Allotments type: `List<(int subjectId, int? subjectOptionId, List<int> teacherIds)>`. Permission check (Timetable / Teacher Allotment edit) before save. Shows loading, success, or error via `ITeacherAllotmentView`. |
| Repository | `lib/repositories/class_subject_repository.dart` | `getSubjectsByClassGrouped`, `getSectionsSubjectsMinimal`, `saveTeacherAllotments` (payload includes `subject_id`, `subject_option_id`, `teacher_ids` per item). |
| Other | `StaffRepository.listStaffByRole`, `ClassRepository.listClasses`, `RoleRepository.listRolesMinimal` | Classes, roles, and staff-by-role for dropdowns and teacher picker. |

- **Modes:** `TeacherAllotmentMode.viewing` | `TeacherAllotmentMode.editing`.
- **Lifecycle:** Presenter created in view `initState`; view calls `_presenter.dispose()` in its `dispose()`.

### 3.2 Backend (FastAPI – application logic)

| Layer | File | Responsibility |
|-------|------|----------------|
| Routes | `app/api/routes/classes.py` | `GET /{class_id}/subjects?section_id=`, `GET /{class_id}/sections-subjects-minimal`, `PUT /{class_id}/sections/{section_id}/teacher-allotments`. |
| Service | `app/services/class_subject_service.py` | `get_subjects_by_class` (groups by subject + option; for option-subjects returns one entry per option with that option’s teachers; fallback: one entry if has_options but no options created), `get_sections_subjects_minimal` (same per-option grouping), `replace_teacher_allotments` (accepts optional `subject_option_id` per allotment; uniqueness by (subject_id, subject_option_id); validates option via SubjectOptionService and subject has_options; delete then insert ClassSubject rows with `subject_option_id`). `assign_subject_to_class`, `copy_subjects`, `update_class_subject` support `subject_option_id`. |
| Schemas | `app/schemas/class_subject.py` | `TeacherAllotmentItem` (subject_id, subject_option_id?, teacher_ids), `TeacherAssignmentItem` (class_subject_id, teacher_id, subject_option_id?), `SubjectWithTeachers` / `SubjectWithTeachersMinimal` (subject, teachers, options?, subject_option_id?, subject_option?). `TeacherAllotmentReplaceRequest` (allotments), `SubjectWithTeachersListResponse`. |
| Model | `app/models/class_subject.py` | `ClassSubject`: class_id, section_id, subject_id, subject_option_id (nullable), teacher_id (nullable). Dynamic table per school; `subject_option_id` added for per-option teacher assignment. |

---

## 4. API Summary

| Method | Path | Purpose |
|--------|------|--------|
| GET | `/classes/{class_id}/subjects?section_id={section_id}` | Subjects for class–section grouped with teachers. |
| GET | `/classes/{class_id}/sections-subjects-minimal` | Sections and per-section subjects (minimal) for a class. |
| PUT | `/classes/{class_id}/sections/{section_id}/teacher-allotments` | Replace all teacher allotments for that class–section. Body: `{ "allotments": [ { "subject_id", "subject_option_id" (optional), "teacher_ids" } ] }`. Backend enforces unique (subject_id, subject_option_id) and validates option when present. |
| GET | `/staff/by-role/{role_id}` | List staff by role (for teacher picker). |

---

## 5. Permissions & Audit

- **Submodule:** 606 – Teacher Allotment (under Timetable module).
- **Edit:** Presenter checks `PermissionService.canEdit(moduleTimetable, submoduleTeacherAllotment)` before calling save API.
- **Audit:** View uses `submoduleId: ModuleConfig.submoduleTeacherAllotment` where audit/permission helpers are used.

---

## 6. Review

### 6.1 Alignment with project rules

- **Architecture:** UI in Flutter (view, state, API calls); business logic and persistence in FastAPI (validation, replace-all semantics, ClassSubject CRUD). No raw SQL; SQLAlchemy ORM used.
- **Flutter:** Uses theme/text theme, GoRouter for navigation, permission checks and loading around API calls, presenter disposed in view `dispose()`.
- **Backend:** One replace-all endpoint keeps semantics simple; validation (duplicates, existence of subject/staff) in service layer.

### 6.2 Strengths

- Clear separation: view holds UI and pending state; presenter coordinates loading/save and permissions; repository maps to API.
- Single “replace allotments” API avoids partial-update complexity and keeps backend authoritative.
- Minimal API (`sections-subjects-minimal`) reduces round-trips when selecting a class.
- Timetable generation explicitly references teacher allotment in its validation message when subject hours are missing.

### 6.3 Subject options

- **Option-subjects:** When a subject has `has_options: true` and has options (e.g. French, German) in SubjectOption, the API returns **one entry per option**; each has `subject_option_id` and `subject_option` (id, name, etc.). The UI shows one card per option (e.g. “Second Language – French”) and pending state is keyed by `(subjectId, optionId)`. Save sends one allotment per card including `subject_option_id`. Timetable generation uses variables keyed by `(subject_id, subject_option_id)` so each option gets the correct teacher.
- **Fallback:** If a subject has `has_options` but no options created yet, the backend returns one entry with `subject_option_id: null` so the user can still assign at least one teacher.
- **Card title:** If the API omits `subject_option` but sends `subject_option_id`, the UI shows “Subject – Option {id}”.

### 6.4 Minor notes

- **Staff names:** View maintains `_staffNameCache` and can fetch staff by ID for names not in the current role list; consider centralising or reusing if other screens need the same pattern.
- **Save scope:** Save is per section; unsaved changes in other sections are not sent. This is consistent with the current UX (per-section Save).

---

## 7. Related docs

- **Technical / grep reference:** [TEACHER_ALLOTMENT_HOW_IT_WORKS.md](TEACHER_ALLOTMENT_HOW_IT_WORKS.md) – API triggers, frontend/backend file list, data flow, grep-oriented keywords.
