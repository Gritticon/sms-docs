# SMS UI — Architecture Deviation Fix List
**Generated:** 2026-04-16  
**Source:** graphify knowledge graph scan (10,022 nodes, 19,912 edges)  
**Scope:** `sms-ui/lib/` — Flutter/GetX frontend  
**Gate:** All items below block promotion from `testing` → `built`

---

## Architecture Rules (from `modules-and-submodules` arch doc)

Every module in `sms-ui` must have **all three** of:

| # | Rule | File location |
|---|------|---------------|
| R1 | GetX controller | `lib/controllers/{module}_controller.dart` |
| R2 | Repository layer | `lib/repositories/{module}_repository.dart` |
| R3 | Binding file | `lib/bindings/{module}_binding.dart` |
| R4 | Registered in `ModuleConfig` | `lib/config/module_config.dart` |

**Pattern reference:** `audit_log` is the only fully compliant module — use it as template:
- `lib/controllers/audit_log_controller.dart`
- `lib/repositories/audit_log_repository.dart`
- `lib/bindings/audit_log_binding.dart`

---

## DEV-TYPE-1 — Missing Controller (70 views affected)

View file exists in `lib/views/` but no matching `lib/controllers/{name}_controller.dart`.

| View file | Controller needed | Priority |
|-----------|------------------|----------|
| `dashboard_view.dart` | `dashboard_controller.dart` | 🔴 HIGH |
| `watchlist_view.dart` | `watchlist_controller.dart` | 🔴 HIGH |
| `watchlist_detail_view.dart` | `watchlist_detail_controller.dart` | 🔴 HIGH |
| `academic_records_view.dart` | `academic_records_controller.dart` | 🔴 HIGH |
| `student_profiles_view.dart` | `student_profiles_controller.dart` | 🔴 HIGH |
| `student_promotion_view.dart` | `student_promotion_controller.dart` | 🔴 HIGH |
| `timetable_view.dart` | `timetable_controller.dart` | 🔴 HIGH |
| `timetable_settings_templates_view.dart` | `timetable_settings_templates_controller.dart` | 🔴 HIGH |
| `teacher_allotment_view.dart` | `teacher_allotment_controller.dart` | 🔴 HIGH |
| `setup_view.dart` | `setup_controller.dart` | 🔴 HIGH |
| `setup_geofence_view.dart` | `setup_geofence_controller.dart` | 🔴 HIGH |
| `certificates_view.dart` | `certificates_controller.dart` | 🟡 MED |
| `staff_detail_view.dart` | `staff_detail_controller.dart` | 🟡 MED |
| `code_of_conduct_view.dart` | `code_of_conduct_controller.dart` | 🟡 MED |
| `code_of_conduct_add_view.dart` | `code_of_conduct_add_controller.dart` | 🟡 MED |
| `code_of_conduct_case_detail_view.dart` | `code_of_conduct_case_detail_controller.dart` | 🟡 MED |
| `classes_sections_view.dart` | `classes_sections_controller.dart` | 🟡 MED |
| `requests_view.dart` | `requests_controller.dart` | 🟡 MED |
| `payroll_advances_view.dart` | `payroll_advances_controller.dart` | 🟡 MED |
| `session_logs_view.dart` | `session_logs_controller.dart` | 🟡 MED |
| `student_exams_view.dart` | `student_exams_controller.dart` | 🟡 MED |
| `student_detail_view.dart` | `student_detail_controller.dart` | 🟡 MED |
| `setup_early_permission_config_view.dart` | `setup_early_permission_config_controller.dart` | 🟡 MED |
| `achievements_view.dart` | `achievements_controller.dart` | 🟡 MED |
| `departments_view.dart` | `departments_controller.dart` | 🟡 MED |
| `holidays_view.dart` | `holidays_controller.dart` | 🟡 MED |
| `home_view.dart` | `home_controller.dart` | 🟡 MED |
| `mobile_restriction_view.dart` | `mobile_restriction_controller.dart` | 🟡 MED |
| `navigation_view.dart` | `navigation_controller.dart` | 🟡 MED |
| `payroll_processing_view.dart` | `payroll_processing_controller.dart` | 🟡 MED |
| `payroll_workspace_view.dart` | `payroll_workspace_controller.dart` | 🟡 MED |
| `profile_view.dart` | `profile_controller.dart` | 🟡 MED |
| `progress_summary_view.dart` | `progress_summary_controller.dart` | 🟡 MED |
| `question_papers_view.dart` | `question_papers_controller.dart` | 🟡 MED |
| `roles_view.dart` | `roles_controller.dart` | 🟡 MED |
| `roles_departments_view.dart` | `roles_departments_controller.dart` | 🟡 MED |
| `student_attendance_view.dart` | `student_attendance_controller.dart` | 🟡 MED |
| `student_profiles_view.dart` | `student_profiles_controller.dart` | 🟡 MED |
| `subjects_view.dart` | `subjects_controller.dart` | 🟡 MED |
| `timetable_generate_view.dart` | `timetable_generate_controller.dart` | 🟡 MED |
| `timetable_preview_view.dart` | `timetable_preview_controller.dart` | 🟡 MED |
| `timetable_template_edit_view.dart` | `timetable_template_edit_controller.dart` | 🟡 MED |
| `house_tag_view.dart` | `house_tag_controller.dart` | 🟡 MED |
| `house_tag_detail_view.dart` | `house_tag_detail_controller.dart` | 🟡 MED |
| `house_tag_form_view.dart` | `house_tag_form_controller.dart` | 🟡 MED |
| `exams_view.dart` | `exams_controller.dart` | 🟡 MED |
| `exam_schedules_view.dart` | `exam_schedules_controller.dart` | 🟡 MED |
| `staff_profiles_view.dart` | `staff_profiles_controller.dart` | 🟡 MED |
| `substitute_view.dart` | `substitute_controller.dart` | 🟡 MED |
| `widget_picker_view.dart` | `widget_picker_controller.dart` | 🟡 MED |
| `salary_templates_view.dart` | `salary_templates_controller.dart` | 🟡 MED |
| `login_view.dart` | `login_controller.dart` | 🔴 HIGH |

---

## DEV-TYPE-2 — Missing Repository Layer

View exists, controller may exist, but no `lib/repositories/{name}_repository.dart`.

| Module | Repository needed | Notes |
|--------|------------------|-------|
| `code_of_conduct` | `code_of_conduct_repository.dart` | Multi-deviation module |
| `setup_geofence` | `setup_geofence_repository.dart` | Multi-deviation module |
| `departments` | `departments_repository.dart` | Multi-deviation module |
| `student_promotion` | `student_promotion_repository.dart` | |
| `student_attendance` | `student_attendance_repository.dart` | |
| `salary_templates` | `salary_templates_repository.dart` | |
| `payroll_workspace` | `payroll_workspace_repository.dart` | |
| `payroll_advances` | `payroll_advances_repository.dart` | |
| `timetable_preview` | `timetable_preview_repository.dart` | |
| `roles_departments` | `roles_departments_repository.dart` | |
| `holidays` | `holidays_repository.dart` | |
| `timetable_generate` | `timetable_generate_repository.dart` | |
| `staff_detail` | `staff_detail_repository.dart` | |
| `certificates` | `certificates_repository.dart` | |
| `exams` | `exams_repository.dart` | |
| `progress_summary` | `progress_summary_repository.dart` | |
| `teacher_allotment` | `teacher_allotment_repository.dart` | |
| `student_detail` | `student_detail_repository.dart` | |
| `setup_leave_types` | `setup_leave_types_repository.dart` | |
| `timetable_template_edit` | `timetable_template_edit_repository.dart` | |
| `class_edit` | `class_edit_repository.dart` | |
| `classes_sections` | `classes_sections_repository.dart` | |
| `academic_records` | `academic_records_repository.dart` | |
| `achievements` | `achievements_repository.dart` | |

---

## DEV-TYPE-3 — Missing Binding File

Only **1 binding exists** (`audit_log_binding.dart`). Every other module needs one.

A binding wires controller + repository into GetX DI. Without it, controller is never injected and the view crashes at runtime.

**Every module from DEV-TYPE-1 and DEV-TYPE-2 also needs a binding file.**

Binding template (copy `audit_log_binding.dart`, rename):
```dart
import 'package:get/get.dart';
import '../controllers/{module}_controller.dart';

class {Module}Binding extends Binding {
  @override
  List<Bind> dependencies() => [
        Bind.lazyPut(() => {Module}Controller()),
      ];
}
```

---

## DEV-TYPE-4 — Not Registered in ModuleConfig

These modules have views/controllers but are **not listed** in `lib/config/module_config.dart`. Routes calling them will fail at runtime.

| Module | Action |
|--------|--------|
| `audit_log` | Add module ID + submodule IDs to `ModuleConfig` |
| `staff_edit` | Add module ID |
| `earnings_master` | Add module ID |
| `deduction_master` | Add module ID |
| `student_attendance` | Add module ID |
| `watchlist` | Add module ID |
| `role_edit` | Add module ID |
| `exam_schedules` | Add module ID |
| `navigation` | Add module ID |
| `salary_templates` | Add module ID |
| `payroll_processing` | Add module ID |
| `payroll_workspace` | Add module ID |
| `classes_sections` | Add module ID |
| `setup_early_permission_config` | Add module ID |
| `widget_picker` | Add module ID |
| `question_papers` | Add module ID (also API side) |
| `roles_departments` | Add module ID |

---

## DEV-TYPE-5 — API Missing Permission Guards

These **FastAPI route modules** have no permission check decorator. Any authenticated user can call them — security risk before prod.

| Route module | File |
|-------------|------|
| `leave_attachments` | `sms-api/app/routers/leave_attachments.py` |
| `student_code_of_conduct` | `sms-api/app/routers/student_code_of_conduct.py` |
| `student_certificates` | `sms-api/app/routers/student_certificates.py` |
| `subject_hours` | `sms-api/app/routers/subject_hours.py` |
| `support_tickets` | `sms-api/app/routers/support_tickets.py` |
| `timetables` | `sms-api/app/routers/timetables.py` |
| `class_subjects` | `sms-api/app/routers/class_subjects.py` |
| `student_fees` | `sms-api/app/routers/student_fees.py` |
| `session_logs` | `sms-api/app/routers/session_logs.py` |
| `departments` | `sms-api/app/routers/departments.py` |

---

## DEV-TYPE-6 — Open Architecture Decisions (blocking "built")

These items are in `testing` but have **unresolved design decisions** documented in specs. Must be decided before QA sign-off.

| Module | Open decision | Source |
|--------|--------------|--------|
| Audit Logging API | Full snapshots vs action metadata only | audit-log-module.md |
| Audit Logging API | Log retention policy (30d? 90d? indefinite?) | audit-log-module.md |
| Audit Logging API | Log immutability (append-only vs soft-delete?) | audit-log-module.md |
| Reminders API | Delivery channel for v1 (push? in-app? email?) | reminders-spec.md |
| Reminders API | Scheduler architecture (celery beat? cron? db polling?) | reminders-spec.md |

---

## Multi-Deviation Modules (highest risk — fix these first)

These modules have **3 or more** simultaneous deviations. They are the hardest to promote and most likely to cause runtime crashes.

| Module | Missing controller | Missing repo | Missing binding | Not in registry |
|--------|-------------------|-------------|----------------|----------------|
| `code_of_conduct` | ✅ | ✅ | ✅ | ✅ |
| `setup_geofence` | ✅ | ✅ | ✅ | — |
| `departments` | ✅ | ✅ | ✅ | — |
| `staff_profiles` | ✅ | ✅ | ✅ | — |
| `student_attendance` | — | ✅ | ✅ | ✅ |
| `payroll_workspace` | ✅ | ✅ | ✅ | ✅ |
| `roles_departments` | ✅ | ✅ | ✅ | ✅ |
| `certificates` | ✅ | ✅ | ✅ | — |
| `timetable` | ✅ | — | ✅ | — |
| `session_logs` | ✅ | — | ✅ | — |

---

## Summary

| Deviation type | Count |
|---------------|-------|
| Missing controller | ~66 views |
| Missing repository | ~24 modules |
| Missing binding | ~65 modules (all except audit_log) |
| Not in ModuleConfig | 17 modules |
| API missing permission guard | 10 routes |
| Open arch decisions | 5 items |

**Only 1 module is fully compliant: `audit_log`**  
Use it as the canonical pattern for all fixes.
