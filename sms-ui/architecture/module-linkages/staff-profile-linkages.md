---
title: Staff Profile Cross-Module Linkages
type: architecture
project: sms-ui
module: Staff Management — Staff Profiles
last-updated: 2026-03-18
---

# Staff Profile — Cross-Module Linkages

> **Depends on:** `Roles & Departments`, `Teacher Allotment`, `Staff Attendance`, `Timetable`, `Payroll`
> **Affects:** `Teacher Allotment`, `Timetable`, `Dashboard`, `Permission System`

Staff Profile is the central aggregator for all staff-related data in sms-ui. It pulls information from multiple modules to build a complete view of each staff member. It is also the foundational record that many other modules depend on — a staff member must exist before they can be allotted, scheduled, or paid.

---

## Depends On

### `Roles & Departments`
Each staff member is assigned a role and belongs to a department. The profile displays this assignment and the role determines which permissions the staff member has.

**Impact:** If a role is changed or deleted, all staff assigned to that role are affected. Their access and permissions change immediately. Department changes affect profile display and filtering.

---

### `Teacher Allotment`
For staff who are teachers, the profile shows which subjects and classes they are allotted to teach.

**Impact:** Changes to teacher allotments are reflected in the staff profile's allotment section. Removing all allotments from a teacher clears this section on their profile.

---

### `Staff Attendance`
The staff profile displays the attendance history and summary for the staff member — present, absent, and leave records.

**Impact:** Each attendance entry made in the Staff Attendance module appears on the relevant staff member's profile. Corrections or deletions in attendance are reflected on the profile.

---

### `Timetable`
For teachers, the profile includes a schedule view showing their teaching timetable — which classes they teach, at what times.

**Impact:** When the timetable is updated or regenerated, the schedule shown on each teacher's profile is updated accordingly.

---

### `Payroll`
Staff payroll data (salary details, payment history) is associated with the staff profile.

**Impact:** Payroll entries and changes appear in the payroll section of the staff profile.

---

## Affects

### `Teacher Allotment`
A staff member must exist in the system before they can be allotted to a subject. If a staff profile is deactivated or deleted, their allotments become invalid.

**Impact:** Deactivating or removing a staff profile requires reassigning their teacher allotments before a valid timetable can be generated.

---

### `Timetable`
Teacher availability is tied to their staff profile status. An inactive or deleted staff member cannot hold timetable slots.

**Impact:** Removing a staff profile triggers a cascade review of timetable slots assigned to that teacher.

---

### `Permission System & Navigation`
The role assigned on the staff profile determines which modules are visible and which actions are permitted in sms-ui for that user.

**Impact:** Changing a staff member's role changes their navigation menu and available actions immediately on next login.

---

### `Dashboard — Personal Widgets`
The following dashboard personal widgets pull data associated with the logged-in staff member's profile:

| Widget | Data Source via Profile |
|---|---|
| `My Timetable Today` | Timetable linked to this staff member |
| `My Attendance` | Staff Attendance linked to this profile |
| `My Classes Today` | Teacher Allotment linked to this profile |
| `My Subjects` | Teacher Allotment linked to this profile |

**Impact:** Any change to the underlying modules (timetable, attendance, allotment) is reflected in these widgets on the next data fetch.

---

## Change Impact Guide

When making changes to **Staff Profiles**, check:

- [ ] Was a staff member's role changed? → Their navigation and permissions change immediately; check they still have access to what they need
- [ ] Was a staff profile deactivated? → Check their `Teacher Allotment` and reassign their subjects/classes
- [ ] Was a staff profile deactivated? → Check `Timetable` for orphaned slots assigned to this teacher
- [ ] Was a staff profile deactivated? → Check `Substitute` if they have upcoming scheduled substitutions
- [ ] Was a new staff member added? → They must be assigned a role via `Roles & Departments` to gain any access
- [ ] Was department assignment changed? → Verify profile display and any department-based filters elsewhere

---

## Related Documents

- `sms-ui/architecture/module-linkages/timetable-linkages.md`
- `sms-ui/architecture/module-linkages/teacher-allotment-linkages.md`
- `sms-ui/architecture/module-linkages/attendance-linkages.md`
- `sms-ui/architecture/module-linkages/roles-departments-linkages.md`
- `sms-ui/modules/built/staff-profiles-how-it-works.md`
