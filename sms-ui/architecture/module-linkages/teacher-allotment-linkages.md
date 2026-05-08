---
title: Teacher Allotment Cross-Module Linkages
type: architecture
project: sms-ui
module: Schedule Management — Teacher Allotment
last-updated: 2026-03-18
---

# Teacher Allotment — Cross-Module Linkages

> **Depends on:** `Staff Profiles`, `Subjects`, `Classes & Sections`
> **Affects:** `Timetable`, `Staff Profile`

Teacher allotment is the bridge between the school's staff and its class/subject structure. It defines which teacher is responsible for which subject in which class. It is a prerequisite for timetable generation and affects what is shown on each teacher's profile.

---

## Depends On

### `Staff Profiles`
Only active staff members can be allotted as teachers. The allotment module pulls the list of available staff from Staff Profiles.

**Impact:** If a staff member is deactivated or removed, their existing allotments become invalid. Allotments must be reassigned before the timetable can be regenerated.

---

### `Subjects`
Allotments are made for specific subjects. Subjects must be created and assigned to classes before they can be allotted to a teacher.

**Impact:** If a subject is removed from a class, all teacher allotments for that subject in that class are invalidated. Timetable must be reviewed after.

---

### `Classes & Sections`
Allotments are scoped to a class and section. The class structure must be in place before allotments can be created.

**Impact:** If a class or section is deleted, all allotments for that class are lost and must be recreated if the class is re-added.

---

## Affects

### `Timetable`
The timetable generation algorithm reads teacher allotments to decide which teacher appears in each slot. Without a valid allotment, a subject cannot be scheduled in the timetable.

**Impact:** Any change to teacher allotments — adding, removing, or changing a teacher for a subject — requires the timetable to be reviewed and potentially regenerated.

---

### `Staff Profile`
Each teacher's profile shows the subjects and classes they are allotted to. This information is derived from the teacher allotment records.

**Impact:** When a teacher is allotted to a new subject or removed from one, their profile's allotment section reflects the change immediately.

---

## Change Impact Guide

When making changes to **Teacher Allotment**, check:

- [ ] Was a teacher replaced for a subject? → Timetable slots for that subject must be reviewed
- [ ] Was a new allotment added? → Check if it creates a scheduling conflict in the timetable
- [ ] Was an allotment removed? → Subject will have no teacher; timetable slot becomes unassigned
- [ ] Was a subject removed from allotment? → Timetable must be regenerated for that class
- [ ] Do affected teachers' profiles show the correct allotments?
- [ ] Does the dashboard `My Subjects` and `My Classes Today` widget reflect the updated allotment?

---

## Related Documents

- `sms-ui/architecture/module-linkages/timetable-linkages.md`
- `sms-ui/architecture/module-linkages/staff-profile-linkages.md`
- `sms-ui/architecture/module-linkages/subjects-linkages.md`
