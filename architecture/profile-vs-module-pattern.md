---
title: "Profile vs Module Pattern"
type: architecture
project: platform
last-updated: 2026-04-26
---

# Profile vs Module Pattern

## Core Rule

Every data domain in the platform has two access points:

| Access Point | Purpose | Who uses |
|-------------|---------|---------|
| **Module / Submodule** (navigation) | Overall / aggregate view — all records, all staff, all students | Admin, management |
| **Profile** (staff or student profile) | Context-specific view — scoped to that individual, that period, that class | The individual themselves |

These are **two lenses on the same data**, not two separate systems.

---

## Why This Pattern Exists

- Staff (especially teachers) should not need module-level permissions to do their job
- Navigation modules are for admin oversight and bulk management
- Profile views surface only what is relevant to that person's context
- Reduces cognitive load — teacher sees their timetable, not all timetables

---

## Examples

### Timetable
| Module view | Profile view |
|-------------|--------------|
| Admin sees all timetables for all classes and staff | Teacher sees only their own timetable on their profile |
| Admin can edit, generate, publish | Teacher navigates by date, taps a period to open workspace |

### Attendance
| Module view | Profile view |
|-------------|--------------|
| Admin sees attendance across all classes, all staff, all dates | Teacher marks attendance for their specific period from their profile timetable |
| Aggregate reports, exception handling, audit | Scoped to that class/section/period only |

### Staff Profile (general)
| Module view | Profile view |
|-------------|--------------|
| Admin views and manages all staff profiles | Staff member views and manages their own profile — leaves, payslips, approvals |

---

## Rules for All Modules

This pattern applies to **every module** on the platform:

1. **Module/submodule** = bird's-eye view for admin/management
2. **Profile** = first-person contextual view for the individual

When building any new feature, ask:
- Does an admin need to see this across the whole school? → Submodule
- Does an individual need to see/act on this for themselves? → Profile view

Never force a teacher or parent to navigate a management module to do their daily tasks.

---

## Applies To

- Timetable
- Attendance (student and staff)
- Leave management
- Payroll / payslips
- Session logs
- Exam schedules and marks
- Homework, diary, prerequisites (teacher period flow)
- Any future module added to the platform

---

## Related Docs

- `docs/sms-ui/modules/needs-spec/teacher-period-flow.md` — first full implementation of this pattern for teachers
- `docs/product/feature-matrix.md` — module status across projects
- `docs/architecture/system-architecture.md` — overall system design
