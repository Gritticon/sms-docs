---
title: Student Profile Cross-Module Linkages
type: architecture
project: sms-ui
module: Student Management — Student Profiles
last-updated: 2026-03-18
---

# Student Profile — Cross-Module Linkages

> **Depends on:** `Classes & Sections`, `Student Attendance`, `Class Attendance`, `Exams & Results`, `Achievements`, `Certificates`, `Documents`, `Code of Conduct`, `House Tag`, `Requests`
> **Affects:** `KYC Student Profile`, `KYC Attendance`

Student Profile is the primary aggregator for all student-related data in sms-ui. It presents a unified view of a student by pulling data from across multiple modules. Unlike Staff Profile, most of the data flows into the student profile from other modules rather than the profile itself being a prerequisite.

> **Important distinction — Student Attendance vs Class Attendance:**
> - **Student Attendance** records the individual student's own attendance (present/absent per day). Shown on the student's own profile tab.
> - **Class Attendance** records attendance for all students in a class, taken by the teacher per session. This is a class-level view and does not feed into the individual student profile — it is accessed via the Class Management module.

---

## Depends On

### `Classes & Sections`
A student must be assigned to a class and section. This is the foundational record — without a class assignment, many other modules cannot associate data with the student correctly.

**Impact:** If a student's class assignment changes (promotion, transfer), all module data tied to that class context (subjects, class attendance, timetable) must be reviewed for accuracy.

---

### `Student Attendance`
The student profile includes an attendance tab showing the student's individual attendance history — present, absent, late, and leave records.

**Impact:** Every attendance entry recorded in the Student Attendance module appears on the student's profile. Corrections or deletions are reflected immediately.

---

### `Exams & Results`
Exam results and academic records for the student are shown in the profile's academic records tab. This includes marks, grades, and report card references.

**Impact:** When results are entered or updated in the Exam Management module, the student's academic records tab on their profile reflects the change.

---

### `Achievements`
Student achievements recorded in the Achievements submodule appear on the student's profile.

**Impact:** Adding or removing an achievement updates the profile's achievement section.

---

### `Certificates`
Certificates issued to the student are accessible from their profile.

**Impact:** When a certificate is issued or revoked, the student's profile certificate section is updated.

---

### `Documents`
Documents uploaded for a student (medical records, consent forms, etc.) are associated with and accessible from the student's profile.

**Impact:** Document uploads or deletions are reflected on the profile.

---

### `Code of Conduct`
Code of conduct incidents recorded against a student appear in their profile's conduct tab.

**Impact:** New entries or resolutions in the Code of Conduct submodule update the student's profile conduct section.

---

### `House Tag`
A student's assigned house tag is displayed on their profile.

**Impact:** Changing a student's house assignment updates the profile display and any house-based groupings or filters.

---

### `Requests`
Requests raised by parents through KYC on behalf of the student appear in the profile's requests section for staff to view and action.

**Impact:** New requests from KYC appear in the student profile. Staff responses made in sms-ui are sent back and visible to the parent in KYC.

---

## Affects

### `KYC — Student Profile`
The parent and student view in the KYC app is an aggregated read of the same student data — profile details, attendance, achievements, certificates, documents, code of conduct. All changes made via sms-ui modules flow through sms-api and are reflected in the KYC student profile view.

**Impact:** Any update to student data in sms-ui — across any of the dependent modules — is visible to the parent/student in KYC on their next data fetch.

---

### `KYC — Attendance`
The individual student attendance records shown in the KYC app come from the same Student Attendance data displayed on the student profile in sms-ui.

**Impact:** Attendance corrections or additions in sms-ui are reflected in the KYC parent attendance view.

---

## Change Impact Guide

When making changes to **Student Profiles** or any module that feeds into it, check:

- [ ] Was a student's class changed? → Review subject assignments, class attendance records, and timetable context for accuracy
- [ ] Was attendance added or corrected? → Verify the profile attendance tab shows the correct summary; verify KYC parent view reflects the correction
- [ ] Was a result entered or updated? → Verify the profile academic records tab is accurate; verify KYC marks view is updated
- [ ] Was an achievement, certificate, or document added/removed? → Verify profile and KYC student profile both reflect the change
- [ ] Was a code of conduct entry added? → Confirm it appears on the profile conduct tab
- [ ] Was a request responded to? → Confirm the response is visible to the parent in KYC

---

## Related Documents

- `sms-ui/architecture/module-linkages/attendance-linkages.md`
- `sms-ui/architecture/module-linkages/subjects-linkages.md`
- `kyc/architecture/module-linkages/student-profile-linkages.md`
- `architecture/data-flow.md`
- `sms-ui/modules/built/staff-profiles-how-it-works.md`
