# Cursor Fix Prompt — SMS UI Architecture Deviations

Paste the block below directly into Cursor chat.

---

```
You are fixing architecture deviations in the sms-ui Flutter project.
Working directory: /Users/mahesh/Documents/work/Project/SMS/sms-ui

READ FIRST (mandatory before writing any code):
1. lib/config/module_config.dart — understand the module registry structure
2. lib/controllers/audit_log_controller.dart — canonical controller pattern
3. lib/repositories/audit_log_repository.dart — canonical repository pattern
4. lib/bindings/audit_log_binding.dart — canonical binding pattern
5. docs/arch-deviations-fix-list.md — full list of all deviations to fix

ARCHITECTURE RULES (enforce strictly):
- Every module needs: controller + repository + binding + ModuleConfig entry
- Controllers: extend GetxController, inject repository via constructor or Get.find
- Repositories: inject ApiService via Get.find<ApiService>(), return ApiResponse<T>
- Bindings: extend Binding, use Bind.lazyPut for controller (and repository if stateful)
- ModuleConfig: add module ID constant + SubmoduleMetadata entries

---

PHASE 1 — Fix multi-deviation modules first (highest QA risk)
Work on these IN ORDER. Complete each module fully (ctrl + repo + binding + registry) before moving to next.

MODULE 1: code_of_conduct
  Views: lib/views/code_of_conduct_view.dart, code_of_conduct_add_view.dart, code_of_conduct_case_detail_view.dart
  Create:
    - lib/controllers/code_of_conduct_controller.dart
    - lib/repositories/code_of_conduct_repository.dart
    - lib/bindings/code_of_conduct_binding.dart
  Update: lib/config/module_config.dart — add moduleCodeOfConduct constant + submodules

MODULE 2: setup_geofence
  View: lib/views/setup_geofence_view.dart
  Create:
    - lib/controllers/setup_geofence_controller.dart
    - lib/repositories/setup_geofence_repository.dart
    - lib/bindings/setup_geofence_binding.dart

MODULE 3: departments
  View: lib/views/departments_view.dart
  Create:
    - lib/controllers/departments_controller.dart
    - lib/repositories/departments_repository.dart
    - lib/bindings/departments_binding.dart

MODULE 4: payroll_workspace
  View: lib/views/payroll_workspace_view.dart
  Create:
    - lib/controllers/payroll_workspace_controller.dart
    - lib/repositories/payroll_workspace_repository.dart
    - lib/bindings/payroll_workspace_binding.dart
  Update: lib/config/module_config.dart — add to registry

MODULE 5: roles_departments
  View: lib/views/roles_departments_view.dart
  Create:
    - lib/controllers/roles_departments_controller.dart
    - lib/repositories/roles_departments_repository.dart
    - lib/bindings/roles_departments_binding.dart
  Update: lib/config/module_config.dart — add to registry

MODULE 6: certificates
  View: lib/views/certificates_view.dart
  Create:
    - lib/controllers/certificates_controller.dart
    - lib/repositories/certificates_repository.dart
    - lib/bindings/certificates_binding.dart

MODULE 7: staff_profiles
  View: lib/views/staff_profiles_view.dart
  Create:
    - lib/controllers/staff_profiles_controller.dart
    - lib/repositories/staff_profiles_repository.dart
    - lib/bindings/staff_profiles_binding.dart

MODULE 8: student_attendance
  View: lib/views/student_attendance_view.dart
  Create:
    - lib/repositories/student_attendance_repository.dart
    - lib/bindings/student_attendance_binding.dart
  Update: lib/config/module_config.dart — add to registry

---

PHASE 2 — High-priority single-deviation modules (controllers only)

For each view below, create the matching controller + binding.
Pattern: copy audit_log_controller.dart, rename class and repository reference.

Views needing controller + binding:
  dashboard_view.dart         → dashboard_controller.dart + dashboard_binding.dart
  watchlist_view.dart         → watchlist_controller.dart + watchlist_binding.dart
  watchlist_detail_view.dart  → watchlist_detail_controller.dart + watchlist_detail_binding.dart
  academic_records_view.dart  → academic_records_controller.dart + academic_records_binding.dart
  student_profiles_view.dart  → student_profiles_controller.dart + student_profiles_binding.dart
  student_promotion_view.dart → student_promotion_controller.dart + student_promotion_binding.dart
  timetable_view.dart         → timetable_controller.dart + timetable_binding.dart
  teacher_allotment_view.dart → teacher_allotment_controller.dart + teacher_allotment_binding.dart
  session_logs_view.dart      → session_logs_controller.dart + session_logs_binding.dart
  login_view.dart             → login_controller.dart + login_binding.dart

---

PHASE 3 — ModuleConfig registry gaps

In lib/config/module_config.dart, add module ID constants and ModuleDefinition entries for:
  audit_log, staff_edit, earnings_master, deduction_master, student_attendance,
  watchlist, role_edit, exam_schedules, navigation, salary_templates,
  payroll_processing, payroll_workspace, classes_sections,
  setup_early_permission_config, widget_picker, question_papers, roles_departments

Follow the existing pattern in the file exactly.

---

PHASE 4 — API permission guards (sms-api)

In each file listed below, add the permission dependency to every router endpoint
that is currently missing it. Look at a compliant router (e.g. audit_log router)
to see the correct pattern, then apply to:

  sms-api/app/routers/leave_attachments.py
  sms-api/app/routers/student_code_of_conduct.py
  sms-api/app/routers/student_certificates.py
  sms-api/app/routers/subject_hours.py
  sms-api/app/routers/support_tickets.py
  sms-api/app/routers/timetables.py
  sms-api/app/routers/class_subjects.py
  sms-api/app/routers/student_fees.py
  sms-api/app/routers/session_logs.py
  sms-api/app/routers/departments.py

---

CONSTRAINTS:
- Do NOT change any existing view files
- Do NOT change any existing model files
- Do NOT modify audit_log_controller.dart, audit_log_repository.dart, audit_log_binding.dart
- Controllers must use the repository — no direct API calls from controllers
- All Rx state must be declared with .obs
- Repositories must return ApiResponse<T> (see audit_log_repository.dart for pattern)
- If a model for a module does not exist yet, create a minimal stub model in lib/models/

AFTER EACH PHASE:
Report back:
  - Files created: list
  - Files modified: list
  - Any module where the existing view contradicts the pattern (note it, do not break the view)

Start with Phase 1, Module 1 (code_of_conduct). Do not proceed to next module until current one is complete.
```
