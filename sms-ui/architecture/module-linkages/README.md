---
title: SMS UI Module Linkages Index
type: architecture
project: sms-ui
last-updated: 2026-03-18
---

# SMS UI — Module Linkages Index

This folder contains one document per module that has significant cross-module dependencies. Each document describes what the module depends on, what it affects, and a change impact checklist.

## How to Use

**To find all modules that depend on a specific module:**
```
grep -r "Depends on.*<ModuleName>" docs/sms-ui/architecture/module-linkages/
```

**To find all modules affected by a specific module:**
```
grep -r "Affects.*<ModuleName>" docs/sms-ui/architecture/module-linkages/
```

**To find the impact guide for a specific module:**
Open the file named `<module-name>-linkages.md` directly.

---

## Module Linkage Files

| File | Module | Depends On | Affects |
|---|---|---|---|
| `timetable-linkages.md` | Timetable | `Classes & Sections`, `Subjects`, `Teacher Allotment`, `Staff Profiles` | `Staff Profile`, `Substitute`, `Dashboard`, `KYC Timetable` |
| `teacher-allotment-linkages.md` | Teacher Allotment | `Staff Profiles`, `Subjects`, `Classes & Sections` | `Timetable`, `Staff Profile` |
| `staff-profile-linkages.md` | Staff Profile | `Roles & Departments`, `Teacher Allotment`, `Staff Attendance`, `Timetable`, `Payroll` | `Teacher Allotment`, `Timetable`, `Dashboard`, `Permission System` |
| `student-profile-linkages.md` | Student Profile | `Classes & Sections`, `Student Attendance`, `Exams & Results`, `Achievements`, `Certificates`, `Documents`, `Code of Conduct`, `House Tag`, `Requests` | `KYC Student Profile`, `KYC Attendance` |
| `subjects-linkages.md` | Subjects | `Classes & Sections` | `Teacher Allotment`, `Timetable`, `Class Attendance`, `Assignments`, `KYC Subjects` |
| `substitute-linkages.md` | Substitute | `Timetable`, `Staff Profiles` | `Timetable (day-level display override)` |
| `attendance-linkages.md` | Student / Staff / Class Attendance | `Student Profiles`, `Staff Profiles`, `Classes & Sections`, `Subjects` | `Student Profile`, `Staff Profile`, `Dashboard`, `KYC Attendance` |
| `roles-departments-linkages.md` | Roles & Departments | _(none — foundational)_ | `Staff Profiles`, `Permission System`, `Navigation`, `All Modules` |
| `class-session-linkages.md` | Class Session | `Classes & Sections`, `Timetable`, `Subjects`, `Staff Profiles` | `KYC Sessions`, `KYC Class Progress` |

---

## Dependency Chain (upstream → downstream)

Reading left to right shows the order in which modules must be set up and the cascade direction of changes:

```
Roles & Departments
      │
      ▼
Staff Profiles ──────────────────────────────────────────────┐
      │                                                       │
      ▼                                                       ▼
Classes & Sections                                     Staff Attendance
      │                                                       │
      ▼                                                       ▼
   Subjects ─────────────────────────────────────────► Staff Profile (view)
      │                    │                   │
      ▼                    ▼                   ▼
Teacher Allotment    Class Attendance     Assignments
      │
      ▼
  Timetable ──────────────────────────────► Substitute (day override)
      │
      ▼
Staff Profile (schedule view) / Dashboard (My Timetable) / KYC Timetable
```

**Student data chain:**
```
Classes & Sections
      │
      ├──► Student Profiles ──► Student Attendance ──► Student Profile (tab) / KYC
      │
      └──► Subjects ──► Assignments / Class Attendance (separate from Student Attendance)
```
