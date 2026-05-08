---
title: Timetable Spec
type: spec
project: kyc
module: Timetable
last-updated: 2026-03-18
---

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Timetable (View Timetable) — Specification

## 1. Summary
The Timetable module lets students and parents view the student’s daily and weekly schedule, including subjects, teachers, rooms, and timings. It replaces static paper timetables with a responsive digital view that can reflect changes made by the school in SMS.

All timetable data and change tracking are handled by the SMS backend; KYC focuses on clear, device-friendly presentation and simple navigation across days and weeks.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display the current day’s timetable for the active student, including periods, subjects, start/end times, and any teacher/room information provided by the backend. | P0 |
| F2 | System shall allow users to switch between days in the current week (e.g., via tabs or date selector). | P0 |
| F3 | System shall allow navigation between weeks (previous/next, and return to current week). | P1 |
| F4 | System shall indicate holidays or days with no classes, based on backend data. | P0 |
| F5 | System shall update the timetable according to the active child for multi-child parents. | P0 |
| F6 | System shall reflect timetable changes made in SMS after data refresh, without requiring app updates. | P0 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Timetable views shall be responsive, presenting an easy-to-read vertical list on mobile and an appropriate layout (e.g., grid) on larger screens. | P0 |
| NF2 | Daily timetable data shall load within 1–2 seconds under normal conditions. | P1 |
| NF3 | Visual design shall make it clear what the current day is and which day/week is being viewed. | P0 |

## 3. User Stories & Acceptance Criteria

### Story 1: Student checks today’s classes
**As a** student, **I want** to quickly see today’s timetable **so that** I know which classes I have and when.

**Acceptance criteria:**
- [ ] Given I am logged in as a student, when I open the Timetable section, then I see today’s periods in chronological order with subjects and times.
- [ ] Given today is a holiday or no-class day, when I open Timetable, then I see a clear message and/or indicator that there are no scheduled classes.

**Requirement IDs:** F1, F4, NF1, NF3

---

### Story 2: Parent reviews child’s weekly schedule
**As a** parent, **I want** to see my child’s weekly timetable **so that** I can plan activities and support them.  

**Acceptance criteria:**
- [ ] Given I am logged in as a parent and have selected a child, when I open Timetable, then I see that child’s timetable for at least the current week with a way to switch days.
- [ ] Given I navigate between days of the week, when I select a different day, then the timetable updates to show that day’s periods.
- [ ] Given I switch to another child, when I open Timetable again, then the timetable updates to that child’s schedule.

**Requirement IDs:** F1, F2, F3, F5, NF1, NF3

---

### Story 3: Reflect timetable changes
**As a** student or parent, **I want** timetable changes made by the school to be reflected in KYC **so that** I always see the current schedule.  

**Acceptance criteria:**
- [ ] Given the school updates timetable data in SMS (e.g., subject swap, room change), when KYC reloads timetable data, then the changes are visible without app update.
- [ ] Given there is a difference between previously loaded timetable and refreshed data, when I view the timetable, then I only see the latest data from SMS without stale conflicting information.

**Requirement IDs:** F1, F3, F6, NF1

## 4. Dependencies
- **SMS API timetable endpoints**: Provide per-student timetable data by day/week, including holidays and changes.
- **Academic calendar configuration**: Defines working days, weekends, and holidays.
- **Student-parent relationships and active child context**: Ensure correct timetable is shown per child.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Frequent timetable changes may confuse users if not clearly indicated. | Medium | Medium | Ensure that the timetable always shows the latest data from SMS and optionally show “Last updated” timestamp if provided. |
| Complex schedules (e.g., rotating timetables) may be hard to present simply. | Medium | Medium | Start with daily/weekly views and rely on SMS to expose a flattened schedule; consider specialized layouts later if needed. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If no timetable is configured for a student (e.g., error or new admission), show a clear message prompting them to contact the school.
- **Invalid input:** If an invalid date or week is requested (e.g., out of academic year), show a generic error and reset to the nearest valid period.
- **Failure/offline:** If timetable data fails to load due to network or backend error, display a retry option and keep any previously loaded timetable on screen if available.

## 7. MVP Scope

### In scope
- Today and current-week timetable views per student.
- Basic daily navigation and optional week navigation.
- Holiday/no-class indication.
- Multi-child parent support.

### Out of scope (backlog)
- Complex rotation or cycle-based timetables beyond linear day/week views.
- Printable or downloadable timetable exports (e.g., PDF).

### Success criteria for MVP
- Students and parents can reliably see accurate daily timetables without relying on printed schedules.
- Reduced timetable-related confusion and fewer timetable clarification requests to school staff after KYC adoption.

