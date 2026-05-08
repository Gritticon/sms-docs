---
title: Subjects Cross-Module Linkages
type: architecture
project: sms-ui
module: Class Management — Subjects
last-updated: 2026-03-18
---

# Subjects — Cross-Module Linkages

> **Depends on:** `Classes & Sections`
> **Affects:** `Teacher Allotment`, `Timetable`, `Class Attendance`, `Assignments`, `KYC — Student Subjects View`

Subjects are the foundational academic unit of the school. They must be defined and assigned to classes before most academic and scheduling modules can function. Subjects sit in the middle of a chain: classes define the structure, subjects fill it, and allotment/timetable/attendance build on top.

---

## Depends On

### `Classes & Sections`
Subjects are assigned to specific classes and sections. A subject cannot exist in the timetable or attendance without being associated with a class.

**Impact:** If a class is deleted, all subjects assigned to that class lose their context. Teacher allotments and timetable slots for those subjects become invalid.

---

## Affects

### `Teacher Allotment`
Teachers are allotted to subjects within a class. A subject must be assigned to a class before it can appear in the allotment process.

**Impact:** Removing a subject from a class removes it from the available allotment options. Any existing allotment for that subject becomes invalid and must be reassigned or removed.

---

### `Timetable`
Timetable slots are built around subjects. Each slot in the timetable represents a subject being taught to a class at a specific time.

**Impact:** Adding a subject makes it eligible for scheduling. Removing a subject invalidates any timetable slots assigned to it — timetable must be reviewed and regenerated.

---

### `Class Attendance`
Class attendance is taken per subject session. The list of subjects available when marking class attendance comes from the subjects assigned to that class.

**Impact:** If a subject is removed from a class, attendance records tied to that subject remain in history but no new attendance can be recorded for it.

---

### `Assignments`
Assignments are created for specific subjects within a class. A subject must exist and be assigned to a class before assignments can be created for it.

**Impact:** Removing a subject removes it from the assignment creation options. Existing assignments linked to that subject remain but may need to be archived or reassigned.

---

### `KYC — Student Subjects View`
Students view their subject list in the KYC app. This list is derived from the subjects assigned to the student's class in sms-ui.

**Impact:** Adding or removing subjects from a class immediately affects what subjects the student sees in KYC.

---

## Change Impact Guide

When making changes to **Subjects**, check:

- [ ] Was a subject added to a class? → It is now available for `Teacher Allotment`, `Timetable` scheduling, and `Assignments`
- [ ] Was a subject removed from a class? → Check `Teacher Allotment` for orphaned allotments for that subject; check `Timetable` for slots assigned to that subject; check `Assignments` for existing assignments
- [ ] Was a subject renamed? → Verify the name is updated consistently across timetable display, attendance records, and KYC student subjects view
- [ ] Was a subject moved from one class to another? → Treat as remove + add; go through all checklist items above for both classes

---

## Related Documents

- `sms-ui/architecture/module-linkages/teacher-allotment-linkages.md`
- `sms-ui/architecture/module-linkages/timetable-linkages.md`
- `sms-ui/architecture/module-linkages/attendance-linkages.md`
