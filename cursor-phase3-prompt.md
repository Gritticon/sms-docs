# Cursor Prompt — Phase 3 (ModuleConfig Registry Gaps)

Paste the block below directly into Cursor.

---

```
You are fixing ModuleConfig registry gaps in the sms-ui Flutter project.
Single file to edit: lib/config/module_config.dart
Working directory: /Users/mahesh/Documents/work/Project/SMS/sms-ui

READ THE FULL FILE FIRST:
  lib/config/module_config.dart

RULES:
  1. Only edit lib/config/module_config.dart — no other files.
  2. Never change existing IDs or rename existing constants.
  3. New submodule ID constants must follow the numbering series of their module.
  4. Every new constant needs a matching SubmoduleMetadata entry in the correct ModuleDefinition.
  5. After edits run: dart analyze lib/config/module_config.dart
     Must return 0 issues.

---

GAPS TO FIX (4 groups):

GROUP 1 — Student Management submodules missing from ModuleDefinition
  Constants already exist — just add SubmoduleMetadata entries inside
  Student Management's submodules list:

  submoduleAcademicRecords = 304   → add: title: 'Academic Records', slug: 'academic-records'
  submoduleStudentExams = 310      → add: title: 'Student Exams',    slug: 'student-exams'
  submodulePromoteStudent = 312    → add: title: 'Promote Student',  slug: 'promote-student'

  Place them in this order within Student Management submodules:
    Student Profiles (301)
    Student Attendance (303)
    Academic Records (304)   ← INSERT HERE
    House Tag (305)
    Certificates (306)
    Documents (307)
    Achievements (308)
    Code of Conduct (309)
    Student Exams (310)      ← INSERT HERE
    Requests (311)
    Promote Student (312)    ← INSERT HERE

---

GROUP 2 — Staff Management: Setup submodule missing from ModuleDefinition
  Constant already exists — add SubmoduleMetadata to Staff Management submodules list:

  submoduleSetup = 508   → add: title: 'Setup', slug: 'setup'

  Place after submoduleRolesDepartments (507) entry.

---

GROUP 3 — Staff Management: Payroll sub-features need new constants + entries
  These are distinct views under the payroll area that have no submodule ID yet.
  Add constants in the submodule ID section under "// Staff Management",
  numbered sequentially after submodulePayrollWorkspace = 509:

  static const int submoduleEarningsMaster   = 510;
  static const int submoduleDeductionMaster  = 511;
  static const int submoduleSalaryTemplates  = 512;
  static const int submodulePayrollProcessing = 513;

  Then add SubmoduleMetadata entries inside Staff Management submodules list,
  after submodulePayrollWorkspace (509):

  submoduleEarningsMaster   → title: 'Earnings Master',    slug: 'earnings-master'
  submoduleDeductionMaster  → title: 'Deduction Master',   slug: 'deduction-master'
  submoduleSalaryTemplates  → title: 'Salary Templates',   slug: 'salary-templates'
  submodulePayrollProcessing → title: 'Payroll Processing', slug: 'payroll-processing'

---

GROUP 4 — Exam Management: Exam Schedule submodule missing from ModuleDefinition
  Constant exists: submoduleExamSchedule = 400 (note: shares same ID as submoduleExams = 400)
  This is intentional — same permission, separate UI entry.
  Add SubmoduleMetadata to Exam Management submodules list after Exams (400):

  submoduleExamSchedule → title: 'Exam Schedule', slug: 'exam-schedule'

  Note: since it shares ID 400 with submoduleExams, this is informational only —
  the permission check resolves to the same permission as Exams.

---

DO NOT add entries for:
  - navigation_view   (system UI shell, not a permission module)
  - widget_picker_view (dashboard config UI, not a permission module)
  - setup_early_permission_config (falls under submoduleSetup = 508 added above)
  - question_papers (submoduleQuestionPapers = 402 already in Exam Management ✅)
  - roles_departments (submoduleRolesDepartments = 507 already in Staff Management ✅)
  - audit_log (moduleAuditLogs = 13 already registered ✅)
  - watchlist (moduleWatchlist = 11 already registered ✅)
  - classes_sections (submoduleClassesSections = 601 already in Class Management ✅)
  - payroll_workspace (submodulePayrollWorkspace = 509 added in Phase 1 ✅)
  - student_attendance (submoduleStudentAttendance = 303 added in Phase 1 ✅)

---

AFTER ALL CHANGES:
Run:
  dart analyze lib/config/module_config.dart

Reply with:
  PHASE3_COMPLETE
  Constants added: [list]
  SubmoduleMetadata entries added: [list with parent module]
  Issues found: 0
```
