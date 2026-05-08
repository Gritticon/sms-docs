---
title: Teacher Allotment — How It Works
type: module
project: sms-ui
module: Schedule Management
last-updated: 2026-03-18
---

# Teacher Allotment – How It Works

Document for developers: API triggers, frontend/backend flow, and grep-friendly reference.

---

## GREP QUICK REFERENCE

| Grep for | Meaning |
|----------|--------|
| `TEACHER_ALLOTMENT_API` | All teacher-allotment–related API paths |
| `TEACHER_ALLOTMENT_FRONTEND` | Flutter entry points, views, presenters, repositories |
| `TEACHER_ALLOTMENT_BACKEND` | FastAPI routes, services, schemas |
| `TEACHER_ALLOTMENT_PERMISSION` | Permission/module IDs |
| `API_GET` / `API_PUT` | HTTP method used |

---

## 1. TEACHER_ALLOTMENT_API – Endpoints Used by Teacher Allotment

### 1.1 Get subjects by class and section (grouped with teachers)

- **API_GET** `GET /classes/{class_id}/subjects?section_id={section_id}`
- **TEACHER_ALLOTMENT_BACKEND** Route: `sms-api/app/api/routes/classes.py` → `get_class_subjects`
- **TEACHER_ALLOTMENT_FRONTEND** Caller: `ClassSubjectRepository.getSubjectsByClassGrouped(classId, sectionId: sectionId)`. Used by `TeacherAllotmentPresenter.loadClassSubjectsData(classId, sectionId)` and `loadClassSubjects(classId, sectionId)`.
- Backend: `ClassSubjectService.get_subjects_by_class(db, school_id, class_id, section_id)`; groups by `(subject_id, subject_option_id)`. For subjects **with** options: one entry per option with `subject_option_id`, `subject_option`, and that option’s teachers; for subjects **without** options: one entry with `subject_option_id` null. If `has_options` but no options created, returns one subject-level entry. Each item: `{ subject, teachers, options?, subject_option_id?, subject_option? }` (teachers include `subject_option_id`).
- Response: `SubjectWithTeachersListResponse` (`data`: list of `SubjectWithTeachers`).

### 1.2 Get sections and per-section subjects (minimal, single call)

- **API_GET** `GET /classes/{class_id}/sections-subjects-minimal`
- **TEACHER_ALLOTMENT_BACKEND** Route: `classes.py` → `get_class_sections_subjects_minimal`
- **TEACHER_ALLOTMENT_FRONTEND** Caller: `ClassSubjectRepository.getSectionsSubjectsMinimal(classId)`; used by `TeacherAllotmentPresenter.loadSectionsSubjectsMinimal(classId)` (e.g. when class is selected before section).
- Backend: `ClassSubjectService.get_sections_subjects_minimal(db, school_id, class_id)`; same per-option grouping as 1.1: one item per option for option-subjects (with `subject_option_id`, `subject_option`); one item per subject for non-option subjects; fallback one item if has_options but no options. Returns sections + per-section subjects (minimal) with teachers.
- Response: `SectionsSubjectsMinimalResponse` (sections, section_subjects).

### 1.3 Replace all teacher allotments for a class and section

- **API_PUT** `PUT /classes/{class_id}/sections/{section_id}/teacher-allotments`
- **TEACHER_ALLOTMENT_BACKEND** Route: `classes.py` → `replace_teacher_allotments`
- **TEACHER_ALLOTMENT_FRONTEND** Caller: `ClassSubjectRepository.saveTeacherAllotments(classId, sectionId, allotments)`; invoked by `TeacherAllotmentPresenter.saveTeacherAllotments(classId, sectionId, allotments)` from `TeacherAllotmentView` on Save. Allotments: `List<(int subjectId, int? subjectOptionId, List<int> teacherIds)>`; payload sends `subject_id`, `subject_option_id` (always, null when no option), `teacher_ids` per item.
- Body: `TeacherAllotmentReplaceRequest` with `allotments`: list of `TeacherAllotmentItem` (`subject_id`, `subject_option_id` optional, `teacher_ids`). Backend: `ClassSubjectService.replace_teacher_allotments(...)`; enforces unique `(subject_id, subject_option_id)`; if `subject_option_id` present, validates via SubjectOptionService and subject `has_options`; deletes existing ClassSubject rows for that class-section, then inserts new rows (one per subject–option–teacher; `subject_option_id` set when provided). Returns updated grouped list.
- Response: `SubjectWithTeachersListResponse`.

### 1.4 List staff by role (teachers for dropdown)

- **API_GET** `GET /staff/by-role/{role_id}`
- **TEACHER_ALLOTMENT_BACKEND** Route: `sms-api/app/api/routes/staff.py` → `list_staff_by_role`
- **TEACHER_ALLOTMENT_FRONTEND** Caller: `StaffRepository.listStaffByRole(roleId)` (e.g. to populate teacher picker in teacher allotment).
- Response: minimal list `[{ "id", "name" }]`.

---

## 2. TEACHER_ALLOTMENT_FRONTEND – Flutter Flow and Files

### 2.1 Module and navigation

- **TEACHER_ALLOTMENT_PERMISSION** Submodule ID: `606` (`ModuleConfig.submoduleTeacherAllotment`). Title: `Teacher Allotment`. Slug: `teacher-allotment`.
- **TEACHER_ALLOTMENT_FRONTEND** Config: `sms/lib/config/module_config.dart` — `submoduleTeacherAllotment = 606`; module entry has `id: submoduleTeacherAllotment`, `title: 'Teacher Allotment'`, `slug: 'teacher-allotment'`.
- **TEACHER_ALLOTMENT_FRONTEND** Navigation: `sms/lib/views/navigation_view.dart` — `case ModuleConfig.submoduleTeacherAllotment: return const TeacherAllotmentView();`; import: `teacher_allotment_view.dart`.

### 2.2 View and modes

- **TEACHER_ALLOTMENT_FRONTEND** View: `sms/lib/views/teacher_allotment_view.dart`. Class: `TeacherAllotmentView`; state: `_TeacherAllotmentViewState`; impl: `_TeacherAllotmentViewImpl` implements `ITeacherAllotmentView`. Pending state keyed by `_pendingKey(subjectId, optionId)` → `"subjectId_optionId"` (optionId `0` for no option). One card per subject or per option; card title “Subject – Option name” or “Subject – Option {id}” when option object missing.
- **TEACHER_ALLOTMENT_FRONTEND** Modes: `TeacherAllotmentMode.viewing`, `TeacherAllotmentMode.editing` (enum `TeacherAllotmentMode`).

### 2.3 Presenter and repository

- **TEACHER_ALLOTMENT_FRONTEND** Presenter: `sms/lib/presenters/teacher_allotment_presenter.dart`. Class: `TeacherAllotmentPresenter`; interface: `ITeacherAllotmentView` (displayClassSubjects, displaySectionSubjects, displaySectionsSubjectsMinimal, showSuccess).
- **TEACHER_ALLOTMENT_FRONTEND** Methods: `loadClassSubjects(classId, sectionId)`, `loadSectionsSubjectsMinimal(classId)`, `loadClassSubjectsData(classId, sectionId)`, `saveTeacherAllotments(classId, sectionId, allotments)`; allotments type: `List<(int subjectId, int? subjectOptionId, List<int> teacherIds)>`.
- **TEACHER_ALLOTMENT_FRONTEND** Repository: `sms/lib/repositories/class_subject_repository.dart`. Methods used: `getSubjectsByClassGrouped(classId, sectionId)`, `getSectionsSubjectsMinimal(classId)`, `saveTeacherAllotments(classId, sectionId, allotments)` (payload: `subject_id`, `subject_option_id`, `teacher_ids` per allotment). Models: `SubjectWithTeachers` and `TeacherAssignmentItem` include `subjectOptionId` / `subjectOption`; parsed from API `subject_option_id`, `subject_option`.

### 2.4 Permission checks

- **TEACHER_ALLOTMENT_PERMISSION** Permission check in presenter: `PermissionService.checkPermission(..., submoduleId: ModuleConfig.submoduleTeacherAllotment)` before save; audit uses `submoduleId: ModuleConfig.submoduleTeacherAllotment`.
- **TEACHER_ALLOTMENT_FRONTEND** View passes `submoduleId: ModuleConfig.submoduleTeacherAllotment` to permission/audit helpers where needed.

---

## 3. TEACHER_ALLOTMENT_BACKEND – FastAPI Flow and Files

### 3.1 Routes

- **TEACHER_ALLOTMENT_BACKEND** Router: `sms-api/app/api/routes/classes.py`; prefix: `/classes`. Endpoints: `GET /{class_id}/subjects`, `GET /{class_id}/sections-subjects-minimal`, `PUT /{class_id}/sections/{section_id}/teacher-allotments`.

### 3.2 Service

- **TEACHER_ALLOTMENT_BACKEND** Service: `sms-api/app/services/class_subject_service.py`. Methods: `get_subjects_by_class(db, school_id, class_id, section_id)` (grouped by subject + option; one entry per option for option-subjects; fallback one entry if has_options but no options), `get_sections_subjects_minimal(db, school_id, class_id)` (same per-option grouping), `replace_teacher_allotments(db, school_id, class_id, section_id, allotments)` (allotments: list of `{ subject_id, subject_option_id?, teacher_ids }`; unique (subject_id, subject_option_id); validates option and has_options when subject_option_id set; verifies subject and staff; delete then insert ClassSubject rows with subject_option_id). Also: `assign_subject_to_class(..., subject_option_id?)`, `copy_subjects`, `update_class_subject` support subject_option_id.

### 3.3 Schemas

- **TEACHER_ALLOTMENT_BACKEND** Schemas: `sms-api/app/schemas/class_subject.py`. `TeacherAllotmentItem`: `subject_id` (int, gt=0), `subject_option_id` (optional), `teacher_ids` (list[int], default []). `TeacherAssignmentItem`: `class_subject_id`, `teacher_id`, `subject_option_id` (optional). `SubjectWithTeachers` / `SubjectWithTeachersMinimal`: `subject`, `teachers`, `options?`, `subject_option_id?`, `subject_option?`. `TeacherAllotmentReplaceRequest`: `allotments` (list[TeacherAllotmentItem], min_length=1). Response: `SubjectWithTeachersListResponse` (data: list of SubjectWithTeachers).

### 3.4 Model

- **TEACHER_ALLOTMENT_BACKEND** Model: `sms-api/app/models/class_subject.py` — ClassSubject (junction: class_id, section_id, subject_id, subject_option_id nullable, teacher_id). Dynamic table per school; used by `replace_teacher_allotments` (ClassSubjectModel from `get_dynamic_model(school_id, ClassSubject, db)`). Timetable generation uses variables keyed by (subject_id, subject_option_id).

### 3.5 Permission

- **TEACHER_ALLOTMENT_PERMISSION** Backend: `sms-api/app/core/permissions.py` — submodule `(606, "Teacher Allotment")`. Routes under `/classes` use staff auth; permission 606 controls access to teacher allotment UI/actions.

---

## 4. Data Flow Summary

1. User opens Teacher Allotment → **TEACHER_ALLOTMENT_FRONTEND** `TeacherAllotmentView`; may load sections+subjects via `getSectionsSubjectsMinimal` or load one section via `getSubjectsByClassGrouped`. Response includes one entry per subject or per option (for option-subjects); each entry has `subject_option_id` and `subject_option` when applicable.
2. User edits subject/option–teacher assignments and taps Save → **TEACHER_ALLOTMENT_FRONTEND** builds `allotments` (list of (subjectId, subjectOptionId?, teacherIds)), calls `TeacherAllotmentPresenter.saveTeacherAllotments` → **TEACHER_ALLOTMENT_API** `PUT .../teacher-allotments` with `TeacherAllotmentReplaceRequest` (each item: subject_id, subject_option_id, teacher_ids).
3. **TEACHER_ALLOTMENT_BACKEND** `replace_teacher_allotments` validates (unique (subject_id, subject_option_id), option valid when set), replaces ClassSubject rows for that class-section (with subject_option_id), returns updated grouped list; **TEACHER_ALLOTMENT_FRONTEND** shows success and can refresh display.

---

## 5. Grep-Oriented Keywords (copy-paste for search)

- `TeacherAllotmentView` `TeacherAllotmentPresenter` `ITeacherAllotmentView` `TeacherAllotmentMode`
- `submoduleTeacherAllotment` `teacher-allotment` `teacher_allotment_view` `teacher_allotment_presenter`
- `saveTeacherAllotments` `getSubjectsByClassGrouped` `getSectionsSubjectsMinimal`
- `_pendingKey` `subjectOptionId` `subject_option_id` `SubjectWithTeachers` `subject_option`
- `replace_teacher_allotments` `TeacherAllotmentItem` `TeacherAllotmentReplaceRequest`
- `teacher-allotments` (URL path segment)
- `ClassSubjectService.replace_teacher_allotments` `ClassSubjectRepository.saveTeacherAllotments`
- `ClassSubject` `subject_option_id` (model column)
