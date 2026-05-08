---
title: Class Session Cross-Module Linkages
type: architecture
project: sms-ui
module: Class Session
last-updated: 2026-03-18
---

# Class Session — Cross-Module Linkages

> **Depends on:** `Classes & Sections`, `Timetable`, `Subjects`, `Staff Profiles`
> **Affects:** `KYC — Sessions View`, `KYC — Class Progress`

Class Session is a live or scheduled teaching session within a class. It is grounded in the timetable structure and subject assignment. The data it generates (session records, progress notes) is surfaced to students and parents in KYC.

---

## Depends On

### `Classes & Sections`
Sessions are conducted for a specific class and section. The class must exist and have students assigned.

**Impact:** If a class is deleted, historical session records remain but no new sessions can be created for it.

---

### `Timetable`
Sessions occur in the context of a scheduled timetable slot. The timetable defines when and which subject is being taught.

**Impact:** If a timetable slot is removed or the timetable is regenerated, sessions that were linked to that slot may need to be reviewed for continuity.

---

### `Subjects`
Each session is tied to a subject. The subject must be assigned to the class.

**Impact:** If a subject is removed from a class, no new sessions can be created for it. Historical session records remain.

---

### `Staff Profiles`
The teacher conducting the session must be an active staff member. Session records are associated with the conducting teacher.

**Impact:** If a staff member is deactivated, they cannot conduct new sessions. Historical session records they conducted remain intact.

---

## Affects

### `KYC — Sessions View`
Students can view their class session records in the KYC app — what was taught, when, and by whom. This data comes from Class Session records in sms-ui.

**Impact:** Session records created or updated in sms-ui are reflected in the KYC sessions view for students in that class.

---

### `KYC — Class Progress`
Progress notes and completion status recorded against a session are visible to students and parents in the KYC Class Progress view.

**Impact:** When a teacher updates class progress in sms-ui, it is visible in KYC on the next data fetch.

---

## Change Impact Guide

When making changes to **Class Session**, check:

- [ ] Was a session created for a class? → Verify the class exists in `Classes & Sections` and the subject is assigned
- [ ] Was a session linked to a timetable slot that was subsequently removed? → Review the session record for validity
- [ ] Was class progress updated? → Verify the KYC Class Progress view reflects the update
- [ ] Was a teacher deactivated who was conducting sessions? → Historical records remain; reassign ongoing sessions to another active teacher

---

## Related Documents

- `sms-ui/architecture/module-linkages/timetable-linkages.md`
- `sms-ui/architecture/module-linkages/subjects-linkages.md`
- `sms-ui/architecture/module-linkages/staff-profile-linkages.md`
- `kyc/architecture/module-linkages/student-profile-linkages.md`
