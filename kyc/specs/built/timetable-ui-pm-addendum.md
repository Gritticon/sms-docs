---
title: Timetable UI PM Addendum
type: spec
project: kyc
module: Timetable
last-updated: 2026-03-18
---

> **Built snapshot** — 2025-03-05 — Timetable UI PM addendum moved to built; acceptance criteria and dependencies confirmed.

# Timetable UI — PM Addendum (Flutter MVP)

Traceable acceptance criteria and dependencies for the KYC Timetable screen. Source: [timetable-ui-build-outline.md](timetable-ui-build-outline.md).

---

## Acceptance Criteria (checklist)


| ID  | Criterion                                                                                                                                                            |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AC1 | On open Timetable, **Day view** for **today** is shown and that day's periods (and breaks when provided) load, or "No classes for this date." / "Holiday" when none. |
| AC2 | User can change date via **day picker**; timetable reloads for selected date; loading indicator visible from request start until response.                           |
| AC3 | Each **period row** shows period label, subject, and time (and teacher/room if provided by API).                                                                     |
| AC4 | **Day view** shows **breaks** between periods (label + time) when API provides them; breaks are visually distinct from period rows.                                  |
| AC5 | Empty state shows correct message when no classes or holiday; error shows AppMessage and retry.                                                                      |
| AC6 | User can open **Week view** and see one week (e.g. Mon–Sun) with a row/card per day; each day shows summary (e.g. period count or first subjects) or "No classes" / "Holiday". |
| AC7 | User can **select a day** in Week view and then see **Day view** for that date.                                                                                     |
| AC8 | User can open **Month view** and see a month calendar with **holidays** marked (when API provides them).                                                             |
| AC9 | User can **select a date** in Month view and then see **Day view** for that date.                                                                                    |
| AC10 | User can switch between **Day**, **Week**, and **Month view** (e.g. toggle in AppBar); selected date is preserved.                                                   |
| AC11 | All labels and copy are student-facing; no teacher-only wording.                                                                                                    |


---

## Dependencies

- **Timetable API (day):** For active student and date, return ordered list of slots (periods and breaks) with label, subject, start/end time, optional teacher/room. Flutter uses stub/mock until API is ready.
- **Timetable API (week):** For active student and a week (date range or start-of-week date), return slots per day or list of days with slots. Enables Week view. Flutter may stub by calling day API per day (max 7) until a dedicated week endpoint exists.
- **Timetable API (month / holidays):** For a month or range, return which dates are holidays (and optionally no-class). Enables Month view overview and holiday markers.
- **Multi-child:** Active child context from session; timetable reloads when parent switches child.

