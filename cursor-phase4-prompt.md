# Cursor Prompt — Phase 4 (API Permission Guards)

Paste the block below directly into Cursor.

---

```
You are adding permission guards to FastAPI route files in the sms-api project.
Working directory: /Users/mahesh/Documents/work/Project/SMS/sms-api

READ THIS FIRST (mandatory — exact pattern to follow):
  app/api/routes/staff_attendance.py

RULES (non-negotiable):
  1. Only edit the route files listed below — no other files.
  2. Do NOT modify schemas, models, services, or middleware.
  3. Import check_staff_permission from app.core.permission_helpers.
  4. Add two module-level constants per file (see pattern in staff_attendance.py).
  5. Call check_staff_permission at the TOP of every endpoint body,
     before any service calls or business logic.
  6. Action string maps to HTTP method:
       GET         → "view"
       POST        → "create"
       PUT / PATCH → "edit"
       DELETE      → "delete"
  7. After each file, run: python -m py_compile app/api/routes/<file>.py
     Must succeed with no output (exit 0).

PATTERN (copy exactly from staff_attendance.py):

  # At top of file, after existing imports:
  from app.core.permission_helpers import check_staff_permission

  # Module-level constants (outside any function):
  <MODULE_NAME>_MODULE_ID = <module_id>
  <MODULE_NAME>_SUBMODULE_ID = <submodule_id>

  # Inside each endpoint function, first line of body:
  check_staff_permission(
      db, auth.school_id, auth.user_id,
      <MODULE_NAME>_MODULE_ID, <MODULE_NAME>_SUBMODULE_ID, "<action>"
  )

  NOTE: auth comes from: auth: RequestContext = Depends(get_staff_auth_context)
  This dependency already exists in all target files — do not re-add it.

---

FILES TO EDIT (7 files):

FILE 1: app/api/routes/leave_attachments.py
  Constants:
    LEAVE_ATTACHMENTS_MODULE_ID = 6
    LEAVE_ATTACHMENTS_SUBMODULE_ID = 506
  Add check_staff_permission to every endpoint.

FILE 2: app/api/routes/student_code_of_conduct.py
  Constants:
    STUDENT_CODE_OF_CONDUCT_MODULE_ID = 4
    STUDENT_CODE_OF_CONDUCT_SUBMODULE_ID = 309
  Add check_staff_permission to every endpoint.

FILE 3: app/api/routes/student_certificates.py
  Constants:
    STUDENT_CERTIFICATES_MODULE_ID = 4
    STUDENT_CERTIFICATES_SUBMODULE_ID = 306
  Add check_staff_permission to every endpoint.

FILE 4: app/api/routes/subject_hours.py
  Constants:
    SUBJECT_HOURS_MODULE_ID = 7
    SUBJECT_HOURS_SUBMODULE_ID = 605
  Add check_staff_permission to every endpoint.

FILE 5: app/api/routes/timetables.py
  Constants:
    TIMETABLES_MODULE_ID = 10
    TIMETABLES_SUBMODULE_ID = 602
  Add check_staff_permission to every endpoint.

FILE 6: app/api/routes/class_subjects.py
  Constants:
    CLASS_SUBJECTS_MODULE_ID = 7
    CLASS_SUBJECTS_SUBMODULE_ID = 605
  Add check_staff_permission to every endpoint.

FILE 7: app/api/routes/departments.py
  Constants:
    DEPARTMENTS_MODULE_ID = 6
    DEPARTMENTS_SUBMODULE_ID = 507
  Add check_staff_permission to every endpoint.

---

DO NOT TOUCH these files (correct auth system, no permission guard needed):
  - app/api/routes/support_tickets.py   → uses internal_employee_bearer (different auth system)
  - app/api/routes/student_fees.py      → module 14, outside PermissionRegistry range (auth-only)
  - app/api/routes/session_logs.py      → module 12, outside PermissionRegistry range (auth-only)

---

AFTER ALL 7 FILES:
Run:
  python -m py_compile \
    app/api/routes/leave_attachments.py \
    app/api/routes/student_code_of_conduct.py \
    app/api/routes/student_certificates.py \
    app/api/routes/subject_hours.py \
    app/api/routes/timetables.py \
    app/api/routes/class_subjects.py \
    app/api/routes/departments.py

Reply with:
  PHASE4_COMPLETE
  Modified: [list all 7 files]
  Endpoints guarded: [count per file]
  Issues found: 0
```
