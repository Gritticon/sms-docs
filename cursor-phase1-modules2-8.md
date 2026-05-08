# Cursor Prompt — Phase 1 Modules 2–8 (Architecture Fixes)

Paste the block below directly into Cursor.

---

```
You are fixing architecture deviations in the sms-ui Flutter/GetX project.
Working directory: /Users/mahesh/Documents/work/Project/SMS/sms-ui

READ THESE FIRST (mandatory — do not skip):
  lib/controllers/audit_log_controller.dart       ← controller pattern
  lib/repositories/audit_log_repository.dart      ← repository pattern  
  lib/bindings/audit_log_binding.dart             ← binding pattern
  lib/controllers/code_of_conduct_controller.dart ← already-fixed reference
  lib/repositories/code_of_conduct_repository.dart

CRITICAL RULES (breaking any = wrong):
  1. Controllers extend GetxController. All mutable state uses .obs.
  2. Repositories inject ApiService or use their own Dio — follow existing repo pattern.
  3. Bindings extend Binding, use Bind.lazyPut(() => XController()).
  4. AppError: NEVER call AppError(message:...) directly.
     Use factories ONLY: AppError.server(), AppError.network(), AppError.fromException()
  5. Do NOT modify any existing view, model, or repository file.
  6. Do NOT create models — all models already exist (listed per module below).
  7. Run `dart analyze` on every file you create. Fix all errors before moving on.

---

For each module below, create exactly these files:
  lib/controllers/{module}_controller.dart
  lib/bindings/{module}_binding.dart

For modules marked "ALSO NEEDS REPO" also create:
  lib/repositories/{module}_repository.dart

After creating all files for a module, run:
  dart analyze lib/controllers/{module}_controller.dart lib/bindings/{module}_binding.dart

Report: files created + 0 issues confirmed, then move to next module.

---

MODULE 2: setup_geofence
  View: lib/views/setup_geofence_view.dart
  Existing repo to delegate to: lib/repositories/school_settings_repository.dart
  Models: (no dedicated model — uses raw geofence settings map from school settings)
  
  Controller state to manage:
    RxBool enabled
    RxDouble latitude, longitude, radius
    RxBool loading
    RxnString errorMessage
  
  Controller methods:
    loadGeofenceSettings()  → calls school_settings_repository to fetch geofence config
    saveGeofenceSettings({required bool enabled, required double lat, required double lng, required double radius})
    → on success: update Rx state, show AppMessage.showSuccess
    → on error: AppMessage.showError
  
  Binding: Bind.lazyPut(() => SetupGeofenceController())
  NO new repo needed — delegate to SchoolSettingsRepository().

---

MODULE 3: departments
  View: lib/views/departments_view.dart
  Existing repo to delegate to: lib/repositories/department_repository.dart
  Models: lib/models/department_model.dart (class Department already exists)
  
  Controller state:
    RxList<Department> departments
    RxList<Department> filtered     ← client-side filter of departments
    RxBool loading
    RxnString errorMessage
    RxString searchQuery
    RxString sortOption             ← 'nameAsc' | 'nameDesc' | 'dateCreatedAsc' | 'dateCreatedDesc'
  
  Controller methods:
    refresh()                       → fetch from department_repository
    search(String query)            → filter departments list client-side
    sort(String option)             → sort filtered list
    createDepartment(...)           → delegate to repo, insert into list
    updateDepartment(...)           → delegate to repo, update in list  
    deleteDepartment(int id)        → delegate to repo, remove from list
  
  Binding: Bind.lazyPut(() => DepartmentsController())
  NO new repo needed — delegate to DepartmentRepository().

---

MODULE 4: payroll_workspace   [ALSO NEEDS REPO]
  View: lib/views/payroll_workspace_view.dart
  Existing repos to delegate to:
    lib/repositories/payroll_repository.dart
    lib/repositories/salary_template_repository.dart
    lib/repositories/department_repository.dart
    lib/repositories/staff_repository.dart
  Models used by view:
    lib/models/payroll_run_model.dart
    lib/models/payslip_model.dart
    lib/models/staff_model.dart
    lib/models/salary_template_model.dart

  Create lib/repositories/payroll_workspace_repository.dart:
    → Thin facade over the 4 repos above
    → Exposes: fetchPayrollRuns(), fetchPayslips(int runId), fetchStaffForPayroll(), fetchSalaryTemplates()
    → Each method delegates to the matching existing repo
    → Returns ApiResponse<T> following audit_log_repository pattern
  
  Controller state:
    RxList<PayrollRun> payrollRuns
    Rxn<PayrollRun> selectedRun
    RxBool loading
    RxnString errorMessage
  
  Controller methods:
    refresh()     → load payroll runs
    selectRun(PayrollRun run)
    runPayroll(...)
  
  Binding: Bind.lazyPut(() => PayrollWorkspaceController())
  Update lib/config/module_config.dart: add submodule entry for payroll_workspace if missing.

---

MODULE 5: roles_departments
  View: lib/views/roles_departments_view.dart
  Existing repos to delegate to:
    lib/repositories/department_repository.dart
    lib/repositories/role_repository.dart
  Models:
    lib/models/department_model.dart
    lib/models/role_model.dart
  
  Controller state:
    RxList<Department> departments
    RxList<dynamic> roles          ← use Role model from role_model.dart
    RxBool loadingDepartments
    RxBool loadingRoles
    RxnString errorMessage
  
  Controller methods:
    refresh()                       → load both departments and roles in parallel using Future.wait
    createDepartment(...), updateDepartment(...), deleteDepartment(int id)
    createRole(...), updateRole(...), deleteRole(int id)
  
  Binding: Bind.lazyPut(() => RolesDepartmentsController())
  NO new repo needed.
  Update lib/config/module_config.dart: add moduleRolesDepartments constant + ModuleDefinition if missing.

---

MODULE 6: certificates
  View: lib/views/certificates_view.dart
  Existing repo to delegate to: lib/repositories/student_repository.dart
    → methods: listCertificateRequests(studentId), createCertificateRequest(...)
  Models:
    lib/models/certificate_request_model.dart
    lib/models/student_model.dart (Student)
  
  Controller state:
    RxList<CertificateRequest> certificates   ← use class from certificate_request_model.dart
    RxnInt selectedStudentId
    RxBool loading
    RxnString errorMessage
    RxBool hasMore
  
  Controller methods:
    selectStudent(int studentId)    → sets selectedStudentId, triggers refresh
    refresh()
    loadMore()
    createCertificate({required int studentId, required CertificateRequestCreate data})
    deleteCertificate({required int studentId, required String itemId})
  
  Binding: Bind.lazyPut(() => CertificatesController())
  NO new repo needed — delegate to StudentRepository().

---

MODULE 7: staff_profiles
  View: lib/views/staff_profiles_view.dart
  Note: view already uses lib/controllers/staff_list_controller.dart for the list.
        This controller handles the module-level wrapper state (permissions, load gate, error boundary).
  Existing repos to delegate to:
    lib/repositories/staff_repository.dart
  Models:
    lib/models/staff_model.dart
    lib/models/staff_list_item_model.dart

  Controller state:
    RxBool initialised
    RxBool permissionGranted
    RxnString errorMessage
  
  Controller methods:
    onInit() → check permission via PermissionService, set permissionGranted
    handlePermissionError()
  
  Binding:
    Bind.lazyPut(() => StaffProfilesController())
    Also include StaffListController in the binding:
      Bind.lazyPut(() => StaffListController())  ← already exists, just bind it here
  
  NO new repo needed.

---

MODULE 8: student_attendance
  View: lib/views/student_attendance_view.dart
  Existing repo to delegate to: lib/repositories/student_repository.dart
    → methods: listStudentAttendance(studentId, ...), markAttendance(...)
  Models:
    lib/models/attendance_model.dart
    lib/models/student_model.dart

  Controller state:
    RxnInt selectedClassId
    RxnInt selectedSectionId
    RxList<Student> students
    RxMap<int, List<dynamic>> attendanceByStudent   ← map studentId → attendance records
    RxBool loading
    RxnString errorMessage
    Rx<DateTime> selectedDate
  
  Controller methods:
    selectClass(int classId)
    selectSection(int sectionId)
    selectDate(DateTime date)
    refresh()             → reload attendance for selected class/section/date
    markPresent(int studentId)
    markAbsent(int studentId)
  
  Binding: Bind.lazyPut(() => StudentAttendanceController())
  Update lib/config/module_config.dart: confirm submoduleStudentAttendance = 303 is present;
    if student_attendance is not in any ModuleDefinition.submodules list, add it to Student Management.
  NO new repo needed — delegate to StudentRepository().

---

AFTER ALL 7 MODULES:
Run full analysis:
  dart analyze lib/controllers/ lib/bindings/ lib/repositories/payroll_workspace_repository.dart

Then reply with this exact format:
  PHASE1_MODULES2-8_COMPLETE
  Created: [list all new files]
  Modified: [list any config changes]
  Issues found: [0 or describe]
```
