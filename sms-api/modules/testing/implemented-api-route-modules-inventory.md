---
title: "Implemented SMS API route modules — inventory"
type: planning-stub
project: sms-api
module: API surface
last-updated: 2026-04-25
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

> Route list refreshed 2026-04-25 — 67 route modules (was 45 in 2026-03-21 inventory). KYC, communications, school module access, leave, dashboard, and attendance route files are included. See `kyc-and-communications-api-surface.md` and `school-module-access-api-surface.md` in `needs-spec/` for docs gaps.

## Status

**Needs spec** — Implementation detected in codebase during pipeline sync on 2026-03-21, but no per-module specification exists in the docs pipeline for most domains.

## What was found in code

- **Location:** `sms-api/app/api/routes/` (one Python module per file, all registered in `sms-api/app/main.py` under prefix `/sms-api`).
- **Type:** FastAPI `APIRouter` modules.
- **Route modules (67):** `academic_years`, `attendance`, `attendance_buckets`, `audit_logs`, `auth`, `certificate_attachments`, `class_sessions`, `class_subjects`, `classes`, `communications_announcements`, `communications_messages`, `contact`, `dashboard`, `deduction_masters`, `departments`, `early_permission_config`, `early_permission_requests`, `earning_masters`, `exams`, `health`, `holidays`, `house_tags`, `images`, `internal_employees`, `kyc_announcements`, `kyc_attendance`, `kyc_auth`, `kyc_complaints`, `kyc_diary`, `kyc_fees`, `kyc_leaves`, `kyc_marks`, `kyc_messages`, `kyc_school`, `kyc_student`, `kyc_timetable`, `leave_attachments`, `leave_balances`, `leave_requests`, `leave_types`, `payroll`, `question_papers`, `roles`, `rooms`, `salary_templates`, `school`, `school_module_access`, `sections`, `session_logs`, `staff`, `staff_attendance`, `student_academic_records`, `student_achievements`, `student_certificates`, `student_code_of_conduct`, `student_documents`, `student_fees`, `student_leaves`, `students`, `subject_hours`, `subject_options`, `subjects`, `substitute`, `support_tickets`, `timetable_settings_templates`, `timetables`, `watchlist`.

## What needs to be documented

- [ ] Split or expand this inventory into per-domain specs where the product needs a contract (prioritise parent-facing and compliance-sensitive APIs).
- [ ] Align `docs/modules/needs-spec/*` stubs that describe *future* full platforms (e.g. fee-management, audit-logging) with the **actual** paths and payloads (e.g. `GET/POST /students/{id}/fees`, `GET /audit-logs/`).
- [ ] Permission model per write route (see deviation stubs from the same sync scan).

## Open questions

- Which route groups should stay as a single “inventory” doc vs get dedicated `built/` specs after review?
