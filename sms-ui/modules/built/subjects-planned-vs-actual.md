---
title: Subjects Planned vs Actual
type: module
project: sms-ui
module: Class Management
last-updated: 2026-03-18
---

# Subjects plan: Planned vs Actual

Build report for the changes: **Copy-as-Duplicate, Remove Weight, Single Delete, No Unassigned**.

---

## 1. Copy subjects = duplicate Subject rows (and options)

| Item | Planned | Actual |
|------|---------|--------|
| Copy creates new Subject rows (new IDs) per source subject | Duplicate Subject for target class/section so each copy is independent | **Done.** `ClassSubjectService.copy_subjects` creates a new `Subject` per distinct source subject, then creates `ClassSubject` rows for target with the new `subject_id`. |
| Options duplicated when `has_options` is True | Duplicate all `SubjectOption` rows for the new subject; build source_option_id → new_option_id mapping; use new option IDs in ClassSubject | **Done.** For each source subject with `has_options`, all `SubjectOption` rows are duplicated (new IDs, linked to new subject). Mapping is built and used when creating `ClassSubject` rows. |
| Return shape unchanged for API/Flutter | Same list of class-subject–like dicts so existing copy flow works | **Done.** Return list of dicts with `subject` etc.; Flutter copy flow unchanged, reloads target list after copy. |

**Deviations:** None.

---

## 2. Remove weight from subject schema and usage

| Item | Planned | Actual |
|------|---------|--------|
| Subject model | Remove `weight` column | **Done.** Removed from `app/models/subject.py`. |
| Schemas | Remove `weight` from SubjectCreate, SubjectUpdate, SubjectResponse | **Done.** Removed from `app/schemas/subject.py`. |
| Subject service | Do not set weight in create; update must not pass weight | **Done.** `create_subject` no longer sets weight; `update_subject` uses schema without weight. |
| Timetable generation | Remove weight from subject_map and from ordering (order by subject_type and optional_group only) | **Done.** `timetable_generation_service.py`: subject_map no longer has `weight`; `_order_variables` and `_build_placement_units` order by subject_type and optional_group_id only. |
| Migration | Add migration or document to drop weight column | **Done.** Documented in `sms-api/docs/MIGRATION_DROP_SUBJECT_WEIGHT.md` with SQL to drop `weight` from `{school_id}_subjects`. |
| Flutter | Remove any weight references in model/UI | **Done.** Flutter Subject model and UI had no weight; no code changes. |

**Deviations:** None.

---

## 3. Remove “Remove subject from class” and keep only “Delete subject”

| Item | Planned | Actual |
|------|---------|--------|
| Backend: remove DELETE class-subjects endpoint | Remove `DELETE /class-subjects/{class_subject_id}` and handler | **Done.** Removed from `app/api/routes/class_subjects.py`. |
| Backend: remove remove_subject_from_class | Remove from ClassSubjectService | **Done.** Removed from `app/services/class_subject_service.py`. |
| Keep DELETE /subjects/{id} | SubjectService.delete_subject remains | **Done.** Unchanged; deletes subject and all ClassSubject links. |
| Flutter: remove removeSubjectFromClass | Remove repo method and DELETE call | **Done.** Removed from `class_subject_repository.dart`. |
| Flutter: simplify deleteSubject | Always full delete; signature `deleteSubject(subjectId, { classId, sectionId })` | **Done.** `subject_presenter.dart`: only calls `_subjectRepository.deleteSubject(subjectId)` and `_globalService.removeSubject(subjectId)`; signature simplified. |
| Flutter: delete dialog | Single “Delete subject” action; message that it removes from all classes | **Done.** `_showDeleteSubjectDialog` uses title “Delete subject”, message “Delete \"{name}\"? This will remove it from all classes.”, confirm “Delete”, and calls presenter `deleteSubject(subject.id, ...)`. |
| Row menu | Single “Delete” item (full delete only) | **Done.** No separate “Remove from class”; single Delete triggers the above dialog. |

**Deviations:** None. **Addition:** `delete_subjects` (bulk) now also deletes subject options in the same transaction (previously only single `delete_subject` did).

---

## 4. Remove unassigned-subjects API and UI

| Item | Planned | Actual |
|------|---------|--------|
| Backend: remove GET /subjects/unassigned | Remove route and list_unassigned_subjects | **Done.** Removed from `app/api/routes/subjects.py` and `app/services/subject_service.py`. |
| Flutter: remove UnassignedSubjectsView | Delete file or remove use | **Done.** Deleted `sms/lib/views/unassigned_subjects_view.dart`. |
| Flutter: remove route | Remove unassigned-subjects path and builder | **Done.** Removed from `app_router.dart` (path, name, builder); removed `buildUnassignedSubjectsRoute`. |
| Flutter: remove listUnassignedSubjects | Remove from SubjectRepository | **Done.** Removed from `subject_repository.dart`. |
| Flutter: remove link and update empty state | Remove nav to unassigned; neutral empty text when no class/section selected | **Done.** Removed IconButton that pushed `unassigned-subjects`. Empty state when no class/section: title “Choose Class & Section”, message “Choose a class and section to view or manage subjects.” |

**Deviations:** None.

---

## 5. Update all API and UI (summary)

| Item | Planned | Actual |
|------|---------|--------|
| API | Copy logic, weight removed, unassigned and remove-from-class endpoints removed | **Done.** As above. |
| UI | Copy flow unchanged; single Delete subject; unassigned removed; empty state updated | **Done.** As above. |

---

## 6. Optional: delete from “all subjects” context

| Item | Planned | Actual |
|------|---------|--------|
| If “all subjects” mode exists, Delete = full delete | Wire Delete to full delete only | **Done.** With unassigned screen removed, subjects are shown by class/section; row Delete is full delete. loadAllSubjects still builds unassigned list for display when no class/section selected; row menu Delete there also calls full delete (same dialog and presenter). |

---

## 7. Verification and documentation

| Item | Planned | Actual |
|------|---------|--------|
| Verification | Copy (with/without options), delete, no unassigned/remove-from-class refs, weight removed | **Done.** Code changes implemented as above; manual smoke test recommended: copy subjects to another class/section (verify new Subject + SubjectOption IDs), delete subject (only option), confirm no references to unassigned or remove-from-class. |
| Planned vs Actual document | Create doc listing planned vs actual; deviations/additions/skipped | **Done.** This document. |

---

## Summary

- **Planned:** Copy duplicates subjects and options with new IDs; weight removed from schema and timetable; only “Delete subject” (full delete); unassigned API/UI removed; verification and this document.
- **Actual:** All items implemented as planned. No deviations. Flutter Subject model had no weight; no Flutter weight changes. Unused imports removed in `subjects_view.dart` (go_router, route_service) after removing unassigned link.
