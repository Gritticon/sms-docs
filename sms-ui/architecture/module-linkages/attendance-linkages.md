---
title: Attendance Cross-Module Linkages
type: architecture
project: sms-ui
module: Student Attendance, Staff Attendance, Class Attendance
last-updated: 2026-03-29
---

# Attendance — Cross-Module Linkages

> **Student Attendance depends on:** `Student Profiles`
> **Student Attendance affects:** `Student Profile (individual tab)`, `KYC Attendance`
>
> **Staff Attendance depends on:** `Staff Profiles`
> **Staff Attendance affects:** `Staff Profile (attendance tab)`, `Dashboard — My Attendance widget`
>
> **Class Attendance depends on:** `Classes & Sections`, `Student Profiles`, `Subjects`
> **Class Attendance affects:** `Class-level view in Class Management` _(does not feed into individual Student Profile)_

Attendance in sms-ui covers three distinct records that are separate from each other. Understanding the distinction is critical:

| Type | Scope | Where it appears |
|---|---|---|
| **Student Attendance** | Individual student's daily attendance | Student's own profile tab |
| **Staff Attendance** | Individual staff member's daily attendance | Staff member's own profile tab |
| **Class Attendance** | All students in a class for a session | Class Management — Class Attendance view |

Class Attendance and Student Attendance are **not the same record and do not feed into each other**. A student's individual attendance profile tab shows Student Attendance records only.

---

## Student Attendance

### Depends On

#### `Student Profiles`
Attendance is recorded against a specific student. The student must have an active profile for attendance to be logged.

**Impact:** If a student is removed from the system, their attendance history remains in records but no new attendance can be logged. If a student transfers classes, historical attendance records remain tied to their profile.

---

### Affects

#### `Student Profile — Attendance Tab`
The student's own attendance history (present, absent, late, leave) is displayed on their profile in sms-ui.

**Impact:** Every attendance entry or correction is immediately reflected on the student's profile attendance tab.

---

#### `KYC — Attendance View (Parent)`
Parents view their child's individual attendance in the KYC app. This is sourced from the same Student Attendance records shown on the sms-ui student profile.

**Impact:** Attendance entries and corrections made in sms-ui are reflected in the parent's KYC attendance view on next fetch.

---

## Staff Attendance

> **Architecture note:** Staff Attendance is being redesigned as a multi-source, pipeline-based system. The dependencies below reflect the new architecture. See full plan: `docs/product/planned/staff-attendance-redesign-plan.md`

### Depends On

#### `Staff Profiles`
Attendance is recorded against a specific staff member. The staff member must have an active profile.

**Impact:** If a staff profile is deactivated, their attendance history is preserved but no new entries can be logged.

---

#### `Staff Shift Templates` *(new dependency)*
The processing function uses the staff member's assigned shift template to determine whether a clock-in is on-time, late, or early-exit. If no template is assigned, the school's default template is used.

**Impact:** Changing a staff member's shift template affects how future attendance events are resolved. Historical records are not retroactively re-processed.

---

#### `School Geofence Settings` *(new dependency)*
The geofence coordinates and radius stored in school settings are loaded by the Flutter Shell on login. If geofence is disabled or not configured, the shell does not send geofence events and the system falls back to manual.

**Impact:** Updating the geofence boundary takes effect on the next shell login. Staff already logged in do not pick up the new boundary until they restart the app.

---

#### `Attendance Events` *(new internal dependency)*
The `{school_id}_attendance_events` table stores raw events from all sources (geofence, manual, biometric, face scan). The processing function reads from this table to produce the final resolved `staff_attendance` record.

**Impact:** The final attendance record is always derived from events. Direct writes to `staff_attendance` without a corresponding event are not allowed in the new architecture.

---

### Affects

#### `Staff Profile — Attendance Tab`
Each staff member's attendance history is displayed on their profile.

**Impact:** Attendance entries and corrections are reflected immediately on the staff profile.

---

#### `Dashboard — Staff Attendance Widget` *(updated)*
The dashboard now shows a real-time summary card: present, absent, late, not-marked, and unreviewed exceptions count (across last 7 days). Navigates to the Attendance Review screen.

**Impact:** Any new event or correction updates the dashboard counts on next fetch. Unreviewed exceptions persist in the count until an admin reviews or corrects them.

---

## Class Attendance

### Depends On

#### `Classes & Sections`
Class attendance is taken for a specific class. The class must exist and have assigned students.

**Impact:** If a class is deleted, class attendance records for that class remain in history but no new records can be created.

---

#### `Student Profiles`
Students must be assigned to the class to appear in the class attendance list.

**Impact:** If a student transfers out of a class, they no longer appear in that class's attendance list. Historical records for that class remain.

---

#### `Subjects`
Class attendance can be recorded per subject session. The subject list available when marking class attendance comes from subjects assigned to the class.

**Impact:** Removing a subject from a class removes it from the class attendance subject options. Existing records for that subject remain in history.

---

### Affects

#### `Class Management — Class Attendance View`
The Class Attendance view in sms-ui shows all students in a class and their attendance per session. This is a class-level aggregate view, not tied to individual student profiles.

**Impact:** Attendance marked here is accessible in the Class Management module only and is not duplicated in individual Student Attendance records.

> **Important:** Class Attendance and Student Attendance are maintained as separate records. Marking a student present in Class Attendance does not automatically update their individual Student Attendance record, and vice versa.

---

## Change Impact Guide

When making changes to any **Attendance** module, check:

**Student Attendance:**
- [ ] Was an attendance entry corrected? → Verify the student's profile attendance tab shows the correction; verify the KYC parent attendance view reflects it
- [ ] Was a student removed from the system? → Historical attendance records are preserved; no further entries possible

**Staff Attendance:**
- [ ] Was an attendance event posted? → Processing function should run and update the final `staff_attendance` record; verify staff profile attendance tab reflects it
- [ ] Was a manual correction made on a geofence record? → Confirm audit log captured the override; verify `resolved_source` updated to `manual`
- [ ] Was a staff profile deactivated? → Confirm no new attendance entries can be logged for them
- [ ] Was a shift template changed? → Historical records are unaffected; confirm new events use the updated template
- [ ] Was the school geofence boundary updated? → Staff must restart the app to pick up the new boundary; note this in any comms

**Class Attendance:**
- [ ] Was a student transferred to another class? → They no longer appear in the original class attendance list; historical records remain
- [ ] Was a subject removed from the class? → Confirm it no longer appears as an attendance option; historical records for that subject are preserved
- [ ] Is the teacher marking attendance for the correct class? → Class Attendance records are class-scoped and do not affect Student Attendance profiles

---

## Related Documents

- `sms-ui/architecture/module-linkages/student-profile-linkages.md`
- `sms-ui/architecture/module-linkages/staff-profile-linkages.md`
- `sms-ui/architecture/module-linkages/subjects-linkages.md`
- `kyc/architecture/module-linkages/student-profile-linkages.md`
- `product/planned/staff-attendance-redesign-plan.md` — full implementation plan for the multi-source pipeline
- `product/staff-attendance-location-feature.md` — product context and architecture overview
