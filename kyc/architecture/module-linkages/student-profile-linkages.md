---
title: KYC Student Profile Cross-Module Linkages
type: architecture
project: kyc
module: Student Profile
last-updated: 2026-03-18
---

# KYC Student Profile — Cross-Module Linkages

> **Depends on:** `Attendance`, `Timetable`, `Marks`, `Documents`, `Achievements`, `Certificates`, `Diary`, `Complaints`, `Requests`, `Class Session`
> **Affects:** _(read-only aggregator — does not feed other KYC modules)_
> **Upstream source:** All data originates in `sms-ui` modules via `sms-api`

The KYC student profile is a read-only aggregator. It presents a consolidated view of a student's data to the parent and student. It does not create or modify data — it pulls and displays data from multiple KYC modules, all of which are ultimately sourced from sms-ui via sms-api.

---

## Depends On

### `Attendance (KYC)`
The student profile displays the student's individual attendance summary — present, absent, late, and leave counts. This data comes from sms-ui Student Attendance records.

**Impact:** If attendance records are corrected in sms-ui, the profile attendance section in KYC reflects the update on next fetch. This is **Student Attendance only** — Class Attendance (all students in a class) is not shown here.

---

### `Timetable (KYC)`
The student's timetable is surfaced within or alongside their profile so students know their class schedule. This data comes from the timetable generated in sms-ui.

**Impact:** When the timetable is updated or regenerated in sms-ui, the timetable shown in the KYC student profile reflects the change.

---

### `Marks (KYC)`
Exam results and marks are displayed on the student profile. This data comes from sms-ui Exam Management — Results.

**Impact:** When new results are published in sms-ui, they appear in the KYC student profile marks section.

---

### `Documents (KYC)`
Documents associated with the student (uploaded in sms-ui) are accessible from the student profile.

**Impact:** Document additions or deletions in sms-ui are reflected in the KYC profile documents section.

---

### `Achievements (KYC)`
Student achievements recorded in sms-ui appear on the KYC student profile.

**Impact:** Adding or removing achievements in sms-ui updates the KYC profile accordingly.

---

### `Certificates (KYC)`
Certificates issued to the student in sms-ui are accessible from the KYC student profile.

**Impact:** Certificate issuance or revocation in sms-ui is reflected in KYC.

---

### `Diary (KYC)`
Diary entries written by the parent or student are associated with the student and may be surfaced in or alongside the profile. Unlike other modules, diary entries originate in KYC — not in sms-ui.

**Impact:** Diary entries created in KYC are stored via sms-api and accessible in the student profile context.

---

### `Complaints (KYC)`
Complaints raised by the parent in KYC, or complaints raised by teachers in sms-ui about the student, are accessible in the student profile context.

**Impact:** New complaints (from either side) and status updates from sms-ui are reflected in the KYC view.

---

### `Requests (KYC)`
Requests raised by the parent in KYC on behalf of the student are accessible in the student profile. Staff responses from sms-ui are also visible here.

**Impact:** Request status updates made in sms-ui are reflected in the KYC student profile requests section.

---

### `Class Session (KYC)`
Session records and class progress associated with the student's class are surfaced in the student profile.

**Impact:** Session records and progress updates made in sms-ui are reflected in the KYC view.

---

## Affects

The KYC student profile is a read-only aggregator. It does not feed or affect other KYC modules. Data flows into it from multiple sources; nothing flows out from it to other modules.

---

## Change Impact Guide

When any upstream module data changes (in sms-ui or KYC), check:

- [ ] Was student attendance corrected in sms-ui? → Profile attendance summary in KYC reflects the correction on next fetch
- [ ] Was the timetable updated in sms-ui? → Timetable displayed in KYC student profile is updated
- [ ] Were new exam results published in sms-ui? → Marks appear in KYC student profile marks section
- [ ] Were documents added/removed in sms-ui? → Document list in KYC student profile is updated
- [ ] Was a complaint resolved in sms-ui? → Complaint status visible to parent in KYC is updated
- [ ] Was a request responded to in sms-ui? → Request status and response visible to parent in KYC

---

## Related Documents

- `sms-ui/architecture/module-linkages/student-profile-linkages.md`
- `kyc/architecture/module-linkages/notifications-linkages.md`
- `architecture/data-flow.md`
- `kyc/specs/built/student-profile-spec.md`
- `kyc/specs/built/attendance-spec.md`
