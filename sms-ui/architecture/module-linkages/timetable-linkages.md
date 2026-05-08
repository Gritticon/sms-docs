---
title: Timetable Cross-Module Linkages
type: architecture
project: sms-ui
module: Schedule Management — Timetable
last-updated: 2026-03-18
---

# Timetable — Cross-Module Linkages

> **Depends on:** `Classes & Sections`, `Subjects`, `Teacher Allotment`, `Staff Profiles`
> **Affects:** `Staff Profile`, `Substitute`, `Dashboard`, `KYC Timetable`

The timetable is a downstream consumer of the school's class, subject, and teacher allotment structure. Any change upstream cascades into the timetable. It is also an upstream data source for several views that display schedule information.

---

## Depends On

### `Classes & Sections`
The timetable is organised by class and section. Each timetable slot is assigned to a specific class. Classes and sections must exist and be fully configured before a timetable can be generated.

**Impact:** If a class or section is deleted or renamed, timetable slots assigned to that class become orphaned. Timetable must be reviewed and regenerated.

---

### `Subjects`
Each timetable slot is tied to a subject. Subjects must be assigned to classes before they can appear in the timetable.

**Impact:** If a subject is removed from a class, all timetable slots for that subject in that class become invalid. Timetable requires regeneration.

---

### `Teacher Allotment`
Teacher allotment defines which teacher handles which subject for which class. The timetable generation algorithm uses allotment data to assign teachers to slots.

**Impact:** If a teacher allotment is changed (different teacher assigned to a subject), the timetable slots for that subject must be reviewed. The timetable may need regeneration to reflect the updated allotment.

---

### `Staff Profiles`
Teacher profile data (name, department, availability) is used when displaying timetable slots.

**Impact:** If a staff member's profile is deactivated or removed, their timetable slots become unassigned. Must be resolved via Teacher Allotment before the timetable is valid again.

---

## Affects

### `Staff Profile`
Each teacher's profile contains a schedule tab that shows their personal timetable — which classes they teach, at what times, on which days. This view is derived directly from the timetable.

**Impact:** When the timetable is updated or regenerated, every affected teacher's profile schedule tab reflects the change immediately.

---

### `Substitute`
The substitute module works on top of the timetable. When a substitute is assigned to cover a class, the timetable view for that specific day shows the substitute teacher in place of the regular teacher.

**Impact:** Substitute assignments do not alter the base timetable — they override the display for that day only. When the substitute period ends, the timetable view reverts to the original.

---

### `Dashboard — My Timetable Today (personal widget)`
The personal widget on the dashboard shows the logged-in staff member's own timetable for today. This data is pulled from the timetable module.

**Impact:** Changes to the timetable are reflected in this widget on the next data fetch.

---

### `KYC — Timetable Feature`
Students and parents view their timetable in the KYC app. This data is served from the same timetable stored in sms-api.

**Impact:** When the timetable is published or updated in sms-ui, the KYC timetable view reflects the change. Unpublished timetables should not be visible in KYC.

---

## Change Impact Guide

When making changes to the **Timetable** module, check:

- [ ] Were any classes or sections involved? → Verify `Classes & Sections` data is intact
- [ ] Were any subjects changed? → Verify `Subjects` assignment to classes is intact
- [ ] Were any teacher allotments changed? → Verify `Teacher Allotment` is updated first
- [ ] Was the timetable regenerated? → Check all affected teachers' profile schedule tabs
- [ ] Is a substitute currently active? → Verify substitute display is still correct for today
- [ ] Is the timetable published? → Confirm KYC timetable view reflects the latest version
- [ ] Does the dashboard widget need a refresh? → My Timetable Today widget should show the updated schedule

---

## Related Documents

- `sms-ui/architecture/module-linkages/teacher-allotment-linkages.md`
- `sms-ui/architecture/module-linkages/substitute-linkages.md`
- `sms-ui/architecture/module-linkages/staff-profile-linkages.md`
- `sms-ui/architecture/module-linkages/subjects-linkages.md`
- `kyc/architecture/module-linkages/student-profile-linkages.md`
- `architecture/data-flow.md`
