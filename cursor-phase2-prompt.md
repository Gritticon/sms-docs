# Cursor Prompt — Phase 2 (High-Priority Single-Deviation Modules)

Paste the block below directly into Cursor.

---

```
You are fixing architecture deviations in the sms-ui Flutter/GetX project.
Working directory: /Users/mahesh/Documents/work/Project/SMS/sms-ui

READ THESE FIRST (mandatory):
  lib/controllers/audit_log_controller.dart         ← controller pattern
  lib/bindings/audit_log_binding.dart               ← binding pattern
  lib/controllers/code_of_conduct_controller.dart   ← already-fixed reference
  lib/controllers/departments_controller.dart       ← already-fixed reference

RULES (same as Phase 1 — non-negotiable):
  1. Controllers extend GetxController. All mutable state uses .obs.
  2. Bindings extend Binding, use Bind.lazyPut(() => XController()).
  3. AppError: NEVER use AppError(...) constructor directly.
     Use ONLY: AppError.server(), AppError.network(), AppError.fromException()
     Use inline: e is AppError ? e.userMessage : 'fallback message'
  4. Do NOT modify any existing view, model, or repository file.
  5. Do NOT create new repositories — all repos already exist (listed per module).
  6. After each module, run dart analyze on its files. Fix all errors. Report 0 issues.

For each module, create exactly:
  lib/controllers/{module}_controller.dart
  lib/bindings/{module}_binding.dart

---

MODULE 1: dashboard
  View: lib/views/dashboard_view.dart
  Existing repo: lib/repositories/dashboard_repository.dart
  Models: lib/models/dashboard_layout_model.dart

  Controller state:
    RxBool loading
    RxnString errorMessage
    Rxn<DashboardLayout> layout     ← use DashboardLayout from dashboard_layout_model.dart

  Controller methods:
    onInit() → calls refresh()
    refresh() → fetch dashboard layout from DashboardRepository, assign to layout

  Binding: Bind.lazyPut(() => DashboardController())

---

MODULE 2: watchlist
  View: lib/views/watchlist_view.dart
  Existing repo: lib/repositories/watchlist_repository.dart
  Models: lib/models/watchlist_model.dart
    Classes: Watchlist, WatchlistListResponse

  Controller state:
    RxList<Watchlist> watchlists
    RxBool loading
    RxBool hasMore
    RxnString errorMessage

  Controller methods:
    onInit() → calls refresh()
    refresh()
    loadMore()
    createWatchlist(...)
    deleteWatchlist(int id)

  Binding: Bind.lazyPut(() => WatchlistController())

---

MODULE 3: watchlist_detail
  View: lib/views/watchlist_detail_view.dart
  Existing repo: lib/repositories/watchlist_repository.dart
  Models: lib/models/watchlist_model.dart
    Classes: WatchlistItem, WatchlistItemWithStudent, WatchlistDetailResponse

  Controller state:
    Rxn<WatchlistDetailResponse> detail
    RxList<WatchlistItemWithStudent> items
    RxBool loading
    RxnString errorMessage
    RxnInt watchlistId

  Controller methods:
    loadDetail(int watchlistId) → fetch detail + items from WatchlistRepository
    addStudent(int watchlistId, int studentId)
    removeStudent(int watchlistId, int itemId)

  Binding: Bind.lazyPut(() => WatchlistDetailController())

---

MODULE 4: academic_records
  View: lib/views/academic_records_view.dart
  Existing repos:
    lib/repositories/student_repository.dart   ← listAcademicRecords(studentId)
    lib/repositories/exam_repository.dart
    lib/repositories/exam_schedule_repository.dart
    lib/repositories/class_repository.dart
  Models:
    lib/models/academic_record_model.dart     ← AcademicRecord, AcademicRecordMetadata
    lib/models/exam_model.dart
    lib/models/exam_schedule_model.dart

  Controller state:
    RxnInt selectedStudentId
    RxnInt selectedClassId
    RxnInt selectedSectionId
    RxList<AcademicRecord> records
    RxBool loading
    RxnString errorMessage

  Controller methods:
    selectStudent(int studentId) → triggers refresh()
    refresh() → fetch records from StudentRepository.listAcademicRecords
    clearSelection()

  Binding: Bind.lazyPut(() => AcademicRecordsController())

---

MODULE 5: student_profiles
  View: lib/views/student_profiles_view.dart
  Existing repo: lib/repositories/student_repository.dart
  Models:
    lib/models/student_profiles_model.dart   ← StudentProfileClass
    lib/models/student_model.dart            ← Student
    lib/models/house_tag_model.dart

  Controller state:
    RxList<Student> students
    RxBool loading
    RxnString errorMessage
    RxString searchQuery
    RxnInt selectedClassId
    RxnInt selectedSectionId

  Controller methods:
    onInit() → refresh()
    refresh()
    search(String query)
    selectClass(int classId)
    selectSection(int sectionId)

  Binding: Bind.lazyPut(() => StudentProfilesController())

---

MODULE 6: student_promotion
  View: lib/views/student_promotion_view.dart
  Existing repo: lib/repositories/student_repository.dart
    → uses StudentRepository.listStudentsForPromotion() or similar
    → check the repo for the exact method name before writing
  Models:
    lib/models/student_promotion_model.dart
    lib/models/student_profiles_model.dart

  Controller state:
    RxList<dynamic> students                  ← use correct type from student_promotion_model.dart
    RxBool loading
    RxnString errorMessage
    RxnInt selectedClassId
    RxnInt targetClassId

  Controller methods:
    onInit() → refresh()
    refresh()
    selectSourceClass(int classId)
    selectTargetClass(int classId)
    promoteStudents(List<int> studentIds)

  Binding: Bind.lazyPut(() => StudentPromotionController())

---

MODULE 7: timetable
  View: lib/views/timetable_view.dart
  Existing repos:
    lib/repositories/timetable_repository.dart
    lib/repositories/timetable_settings_template_repository.dart
  Models:
    lib/models/timetable_model.dart
      Classes: Timetable, TimetableWithSlots, TimetableListData, TimetableSlot

  Controller state:
    RxList<Timetable> timetables
    Rxn<TimetableWithSlots> selected
    RxBool loading
    RxnString errorMessage
    RxnInt selectedClassId
    RxnInt selectedSectionId

  Controller methods:
    onInit() → refresh()
    refresh()
    selectTimetable(Timetable t) → load full TimetableWithSlots
    createTimetable(...)
    deleteTimetable(int id)

  Binding: Bind.lazyPut(() => TimetableController())

---

MODULE 8: teacher_allotment
  View: lib/views/teacher_allotment_view.dart
  Existing repos:
    lib/repositories/class_repository.dart
    lib/repositories/role_repository.dart
    lib/repositories/staff_repository.dart
  Models:
    lib/models/class_model.dart
    lib/models/section_model.dart
    lib/models/staff_model.dart
    lib/models/subject_with_teachers_model.dart   ← check exact class name in file
    lib/models/role_model.dart

  Controller state:
    RxnInt selectedClassId
    RxnInt selectedSectionId
    RxList<dynamic> subjectAllocations      ← use correct type from subject_with_teachers_model.dart
    RxBool loading
    RxnString errorMessage

  Controller methods:
    onInit() → refresh()
    selectClass(int classId)
    selectSection(int sectionId)
    refresh() → load subject-teacher allocations for selected class/section
    assignTeacher(int subjectId, int staffId)
    removeTeacher(int subjectId, int staffId)

  Binding: Bind.lazyPut(() => TeacherAllotmentController())

---

MODULE 9: session_logs
  View: lib/views/session_logs_view.dart
  Existing repo: lib/repositories/session_logs_repository.dart
  Models:
    lib/models/session_log_model.dart
      Classes: SessionLog, MySessionPeriod, SessionLogUpsertRequest

  Controller state:
    RxList<SessionLog> logs
    RxBool loading
    RxBool hasMore
    RxnString errorMessage
    Rx<DateTime> selectedDate

  Controller methods:
    onInit() → selectedDate = today, refresh()
    refresh()
    loadMore()
    selectDate(DateTime date) → triggers refresh()
    upsertLog(SessionLogUpsertRequest request)

  Binding: Bind.lazyPut(() => SessionLogsController())

---

MODULE 10: login
  View: lib/views/login_view.dart
  Existing repo: lib/repositories/auth_repository.dart
    → login({required String schoolId, required String identifier, required String password})
    → logout()
    → verifyToken()

  Controller state:
    RxBool loading
    RxnString errorMessage
    RxBool obscurePassword

  Controller methods:
    login({required String schoolId, required String identifier, required String password})
      → on success: navigate to home via Get.offAllNamed('/') or equivalent
      → on failure: set errorMessage
    togglePasswordVisibility()

  Binding: Bind.lazyPut(() => LoginController())

---

AFTER ALL 10 MODULES:
Run:
  dart analyze lib/controllers/ lib/bindings/

Reply with:
  PHASE2_COMPLETE
  Created: [list all 20 new files]
  Issues found: 0
```
