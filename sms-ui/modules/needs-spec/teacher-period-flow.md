---
title: "Teacher Period Flow — SMS-UI"
type: spec
project: sms-ui
module: Teacher Period Flow
last-updated: 2026-04-26
---

# Teacher Period Flow — Specification

## 1. Summary

Zero-permission teacher model. Teachers access everything through their profile timetable. Tapping a period opens a context-aware Period Workspace with 6 blocks: timetable context, classwork, attendance, prerequisites, diary, homework. No standalone module navigation required for teachers.

This follows the **Profile vs Module pattern** — see `docs/architecture/profile-vs-module-pattern.md`.

---

## 2. Atomic Requirements

### 2.1 Functional

| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | Teacher profile shows today's timetable by default | P0 |
| F2 | Teacher can navigate timetable by date (prev/next day) | P0 |
| F3 | Tapping a period opens the Period Workspace for that period | P0 |
| F4 | Period Workspace derives class, section, subject, date from timetable — no manual selection | P0 |
| F5 | If school has attendance enabled → attendance block shown first before other blocks | P0 |
| F6 | Teacher can mark each student present/absent in attendance block | P0 |
| F7 | Attendance block skipped if attendance not required for that period | P0 |
| F8 | Classwork block: teacher can write/edit planned lesson (plan before, update after) | P0 |
| F9 | Classwork is staff-only — not published to KYC | P0 |
| F10 | Diary block: teacher writes what was actually covered | P0 |
| F11 | Diary published to KYC (student/parent can view) | P0 |
| F12 | Homework block: teacher assigns task with optional due date | P0 |
| F13 | Homework published to KYC (student/parent can view) | P0 |
| F14 | Prerequisites block: teacher sets what students must bring/prepare/know for next occurrence of this subject | P0 |
| F15 | Prerequisites published to KYC (student/parent can view) | P0 |
| F16 | Prerequisites set in period N appear in Period Workspace of the NEXT occurrence of same subject/class | P0 |
| F17 | Teacher can navigate prev/next day within Period Workspace to view or set prerequisites for adjacent periods | P1 |
| F18 | All 6 blocks persist per period (period ID + date + class + subject as composite key) | P0 |
| F19 | Teacher can save blocks independently (not one big save) | P1 |

### 2.2 Non-Functional

| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Teacher sees only their own timetable periods — no cross-staff access | P0 |
| NF2 | Attendance data scoped to teacher's assigned class/section only | P0 |
| NF3 | KYC-published blocks (diary, homework, prerequisites) visible only to students enrolled in that class/section | P0 |
| NF4 | Classwork block never exposed via KYC API | P0 |

---

## 3. The 6 Blocks

| Block | Set when | Staff-only | KYC visible |
|-------|----------|------------|-------------|
| **Timetable context** | Derived from timetable | Yes | No |
| **Classwork** | Before + after class | Yes | No |
| **Attendance** | During class | Yes | No (result visible via attendance submodule) |
| **Diary** | During/after class | No | Yes — student/parent |
| **Homework** | After class | No | Yes — student/parent |
| **Prerequisites** | After class (for next occurrence) | No | Yes — student/parent |

---

## 4. User Stories & Acceptance Criteria

### Story 1: Teacher starts their day
**As a** teacher, **I want** to see today's timetable when I open my profile **so that** I know my periods without navigating any modules.

**Acceptance criteria:**
- [ ] Profile default tab shows today's timetable
- [ ] Periods shown in time order with subject, class, section
- [ ] Teacher can navigate to other dates
- [ ] Tapping a period opens Period Workspace

**Requirement IDs:** F1, F2, F3

---

### Story 2: Teacher runs a period with attendance
**As a** teacher, **I want** attendance to appear first when I tap my period **so that** I don't forget to mark it before teaching.

**Acceptance criteria:**
- [ ] If school attendance setting = enabled → attendance block shown first
- [ ] Student list auto-loaded from class/section (no manual select)
- [ ] Teacher marks present/absent per student
- [ ] After saving attendance, remaining blocks unlock
- [ ] If attendance setting = disabled → attendance block hidden, other blocks accessible immediately

**Requirement IDs:** F4, F5, F6, F7

---

### Story 3: Teacher plans and logs classwork
**As a** teacher, **I want** to write my lesson plan before class and update it after **so that** I have a record of what was planned vs done.

**Acceptance criteria:**
- [ ] Classwork block editable before and after the period time
- [ ] Classwork saved per period occurrence (date + period + class + subject)
- [ ] Classwork NOT visible in KYC at any point
- [ ] Teacher can open a past period and view/edit classwork

**Requirement IDs:** F8, F9, F18

---

### Story 4: Teacher writes diary
**As a** teacher, **I want** to write what I actually covered in the class **so that** parents and students can see it in KYC.

**Acceptance criteria:**
- [ ] Diary block editable during and after class
- [ ] Diary published to KYC under the student's diary/class feed for that date
- [ ] Empty diary shows placeholder — not published until teacher saves content
- [ ] Teacher can edit diary for past periods

**Requirement IDs:** F10, F11

---

### Story 5: Teacher assigns homework
**As a** teacher, **I want** to assign homework with an optional due date **so that** students and parents see it in KYC.

**Acceptance criteria:**
- [ ] Homework block has text field + optional due date picker
- [ ] Homework published to KYC for enrolled students
- [ ] If no homework → block can be left empty (no forced entry)
- [ ] Due date defaults to next school day if not set (or left blank)

**Requirement IDs:** F12, F13

---

### Story 6: Teacher sets prerequisites for next class
**As a** teacher, **I want** to set what students must prepare before the next class **so that** students/parents see it in KYC and I see it myself when I open the next period.

**Acceptance criteria:**
- [ ] Prerequisites block editable after current period
- [ ] Prerequisites saved and linked to the NEXT occurrence of same subject + class/section
- [ ] When teacher opens next period workspace → prerequisites appear at top as a reminder
- [ ] Prerequisites published to KYC for enrolled students
- [ ] Teacher can navigate to next/prev day within workspace to check or set prerequisites for adjacent occurrences

**Requirement IDs:** F14, F15, F16, F17

---

## 5. Dependencies

- **Timetable module**: period data (subject, class, section, time slots) must exist per teacher
- **School settings**: attendance-required flag must be configurable per school
- **Student roster**: class/section → enrolled students list (for attendance)
- **KYC publish pipeline**: diary, homework, prerequisites must flow to KYC per student/class scope
- **Subject recurrence logic**: system must identify "next occurrence" of same subject + class to link prerequisites

---

## 6. Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Substitute teacher — whose period workspace shows? | High | Med | Sub sees original period context; attendance credited to sub |
| Teacher edits past diary/homework — KYC shows stale data | Med | High | KYC pulls live from DB; edit updates immediately |
| No timetable set for teacher → blank profile | High | Med | Show empty state with message; guide admin to assign timetable |
| Same subject appears multiple times per week — wrong "next occurrence" linked | High | Med | Next occurrence = nearest future date with same subject + class + teacher |
| Attendance marked twice (teacher + admin override) | Med | Low | Last-write-wins with audit trail |

---

## 7. Edge Cases

- **Empty state — no periods today**: show "No classes scheduled today" with date nav
- **Period in the past**: all blocks still editable (teacher can fill retrospectively)
- **No students in class**: attendance block shows empty list with warning
- **Prerequisites not set**: next period workspace shows "No prerequisites set for this class"
- **Substitute assigned**: sub sees period workspace; original teacher loses edit access for that period
- **School disables a KYC block** (e.g. hides homework): block hidden in KYC but still editable in SMS

---

## 8. MVP Scope

### In scope
- Timetable view on teacher profile (today + date navigation)
- Period Workspace with all 6 blocks
- Attendance block (conditional on school setting)
- Classwork (staff-only), Diary, Homework, Prerequisites
- KYC publish: diary, homework, prerequisites
- Prerequisites linking to next occurrence

### Out of scope (backlog)
- Rich text / media attachments in diary or homework
- Homework submission by student (viewing only in MVP)
- Bulk homework assign across multiple classes
- Push notification to parents when homework/prerequisites updated
- Analytics: class coverage %, homework completion rate

### Success criteria for MVP
- Teacher completes full period flow (attendance → diary → homework → prerequisites) without leaving profile context
- KYC shows correct diary, homework, prerequisites scoped to enrolled students within 30 seconds of teacher saving
