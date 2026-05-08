---
title: Attendance Module
type: module
project: sms-ui
module: Attendance
last-updated: 2026-03-26
---

## Evolution: Multi-Source Attendance System (Planned)

This module is being evolved into a **multi-source, exception-driven attendance system**.

**Key change in mental model:**
- Geofence is the **primary** source — system marks attendance automatically
- Manual is a **fallback/correction** source — not the primary daily action
- Future sources: biometric, face scan

**UX redesign required:**
- Current screen is a manual-marking form (primary action = mark attendance)
- Target screen is an exception-review dashboard (primary action = review anomalies, correct exceptions)

See full product spec: `docs/product/staff-attendance-location-feature.md`

Related specs:
- Shift templates: `docs/sms-ui/modules/needs-spec/staff-shift-templates.md`
- Rules engine: `docs/sms-ui/modules/needs-spec/attendance-rules-engine.md`
- Geofence settings: `docs/sms-ui/modules/needs-spec/geofence-school-settings.md`
- Flutter shell: `docs/sms-ui-shell/overview.md`

Schema changes needed: add `geofence`, `biometric`, `face_scan` to `source` enum. Open decision: introduce separate `attendance_events` table for raw events.

---

# Attendance Module – Documentation

## Grep patterns (copy-paste)

```
# Backend
attendance.py
staff_attendance.py
attendance_service.py
staff_attendance_service.py
staff_attendance.py
/students/attendance
/staff-attendance/
/staff-attendance/init
/staff-attendance/bulk
/audit-logs/

# Flutter
student_attendance_view.dart
staff_attendance_view.dart
attendance_tab.dart
staff_attendance_presenter.dart
staff_attendance_controller.dart
staff_attendance_repository.dart
attendance_model.dart
staff_attendance_model.dart
attendance_audit_log_model.dart
attendance_date_picker.dart
attendance_day_table_view.dart
attendance_filter_chips.dart
attendance_status_chip.dart

# Module IDs
submoduleStudentAttendance 303
submoduleStaffAttendance 504
submoduleClassAttendance 604
moduleStaffManagement 6

# Selection-based (staff)
selectedStaffIds
applyStatusToSelected
onToggleSelection

# Enums / status
AttendanceStatus
AttendanceSource
present absent late half_day holiday notMarked

# API routes
GET /students/attendance
POST /students/attendance
GET /staff-attendance/
GET /staff-attendance/init
GET /staff-attendance/staff/
POST /staff-attendance/bulk
PUT /staff-attendance/
```

---

## Overview

- Class Attendance: by class/section and date. Statuses: Present, Absent, Late, Half Day, Holiday
- Staff Attendance: by date with clock-in/clock-out. Statuses: Present, Absent, Late, Half Day, Holiday, Not Marked
- Dynamic tables: {school_id}_attendance, {school_id}_staff_attendance

---

## Module IDs

- student-attendance: 303 (Student Management)
- staff-attendance: 504 (Staff Management)
- class-attendance: 604 (Class Management)

---

## 1. Class Attendance

### Backend

- Table: {school_id}_attendance
- Model: app/models/attendance.py
- Schema: app/schemas/attendance.py
- Service: app/services/attendance_service.py
- Routes: app/api/routes/students.py

Columns: id, student_id, date, status, remarks, marked_by, created_at, updated_at
Status enum: PRESENT, ABSENT, LATE, HOLIDAY, HALF_DAY

### API

- GET /students/attendance ?page, page_size, student_id, attendance_date
- POST /students/attendance Body: AttendanceCreate (student_id, date, status, remarks?, marked_by?)

### Flutter

- lib/views/student_attendance_view.dart  StudentAttendanceView
- lib/views/student_details/attendance_tab.dart  AttendanceTab
- lib/widgets/attendance_date_picker.dart
- lib/widgets/attendance_filter_chips.dart
- lib/widgets/attendance_summary_compact.dart
- lib/widgets/attendance_student_day_table_view.dart
- lib/models/attendance_model.dart  Attendance, AttendanceMarkRequest, AttendanceMarkItem
- lib/repositories/student_repository.dart  getAttendance, getAttendanceByClassSection, markAttendance, createAttendance

### Flow

1. Select Class + Section
2. Load students
3. Select date, load existing attendance (filtered by class/section)
4. Mark Present/Absent/Late/Half Day
5. Submit: StudentRepository.markAttendance -> POST /students/attendance per student
6. Submitted = read-only

---

## 2. Staff Attendance

**Full documentation**: [STAFF_ATTENDANCE.md](STAFF_ATTENDANCE.md)

### Backend

- Table: {school_id}_staff_attendance
- Model: app/models/staff_attendance.py
- Schema: app/schemas/staff_attendance.py
- Service: app/services/staff_attendance_service.py
- Routes: app/api/routes/staff_attendance.py

Columns: id, staff_id, date, status, reason, check_in_time, check_out_time, source, notes, created_at, updated_at
Index: idx_staff_attendance_staff_date (staff_id, date)
Source: MANUAL, MACHINE

### API

- GET /staff-attendance/ ?date
- GET /staff-attendance/init ?date
- GET /staff-attendance/staff/{id} ?month or ?year&month
- GET /staff-attendance/staff/{id}/absent ?month or ?year&month
- POST /staff-attendance/bulk  Body: BulkAttendanceRequest
- PUT /staff-attendance/{id}  Body: StaffAttendanceUpdate

Permissions: module 6, submodule 504, view, edit

### Flutter

- lib/views/staff_attendance_view.dart  StaffAttendanceView
- lib/presenters/staff_attendance_presenter.dart  StaffAttendancePresenter
- lib/controllers/staff_attendance_controller.dart  StaffAttendanceController
- lib/repositories/staff_attendance_repository.dart  StaffAttendanceRepository
- lib/models/staff_attendance_model.dart  StaffAttendance, AttendanceSummary

Widgets:
- lib/widgets/attendance_date_picker.dart
- lib/widgets/attendance_day_table_view.dart
- lib/widgets/attendance_filter_chips.dart
- lib/widgets/attendance_status_chip.dart
- lib/widgets/attendance_summary_cards.dart
- lib/widgets/attendance_department_section.dart
- lib/widgets/attendance_department_group.dart
- lib/widgets/attendance_staff_item.dart
- lib/widgets/attendance_staff_card.dart
- lib/widgets/attendance_staff_item_enhanced.dart
- lib/widgets/attendance_audit_log_drawer.dart
- lib/widgets/absence_reasons_widget.dart

### Flow

1. GET /staff-attendance/init?date=
2. Group staff by department; row checkboxes + header select-all
3. Selection-based: status chips (Present, Absent, Late, Half Day) + Apply
4. Apply: applyStatusToSelected -> bulk save; Absent requires reason dialog
5. Per-row status tap: auto-save via bulkUpdateAttendance
6. Audit: GET /audit-logs/?module_id=6&submodule_id=504&date=

### Profile integration

- lib/views/staff_detail_view.dart uses StaffAttendancePresenter.loadAttendanceByMonth, loadAbsentRecordsByMonth

---

## 3. Audit logging

- Backend: AuditLogger.log_audit(module="staff_attendance", action=bulk_create|bulk_update|update)
- Flutter: lib/models/attendance_audit_log_model.dart  AttendanceAuditLog
- API: GET /audit-logs/ ?module_id=6&submodule_id=504&date=&staff_id=

---

## 4. Status values

Backend: present, absent, late, holiday, half_day
Flutter Staff: present, absent, late, halfDay, holiday, notMarked
Flutter Student (AttendanceStatusConverter): Present, Absent, Late, HalfDay, Holiday

---

## 5. File list

Backend (sms-api):
- app/models/attendance.py
- app/models/staff_attendance.py
- app/schemas/attendance.py
- app/schemas/staff_attendance.py
- app/services/attendance_service.py
- app/services/staff_attendance_service.py
- app/api/routes/students.py
- app/api/routes/staff_attendance.py
- app/api/routes/audit_logs.py
- app/core/dynamic_tables.py

Flutter (sms):
- lib/models/attendance_model.dart
- lib/models/staff_attendance_model.dart
- lib/models/attendance_audit_log_model.dart
- lib/models/attendance_audit_log_model.dart (shared with staff attendance)
- lib/views/student_attendance_view.dart
- lib/views/staff_attendance_view.dart
- lib/views/student_details/attendance_tab.dart
- lib/views/staff_detail_view.dart
- lib/presenters/staff_attendance_presenter.dart
- lib/controllers/staff_attendance_controller.dart
- lib/repositories/staff_attendance_repository.dart
- lib/repositories/student_repository.dart
- lib/config/module_config.dart

---

## 6. Student vs Staff

- Student: class+section+date, create-only, no bulk, no clock-in/out, remarks optional
- Staff: date only, selection-based (checkboxes + Apply), per-row auto-save, bulk upsert, clock-in/out, reason required when absent, audit logs
