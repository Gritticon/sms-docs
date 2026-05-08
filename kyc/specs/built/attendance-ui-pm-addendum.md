---
title: Attendance UI PM Addendum
type: spec
project: kyc
module: Attendance
last-updated: 2026-03-18
---

> **Built snapshot** — Attendance UI implemented. See **attendance-ui-build-outline.md** in this folder.

# Attendance UI — PM Addendum (Flutter MVP)

Traceable acceptance criteria and dependency for the KYC Attendance screen. Source: [attendance-ui-build-outline.md](attendance-ui-build-outline.md).

---

## Acceptance criteria (checklist)

| ID | Criterion |
|----|-----------|
| AC1 | On open Attendance, the current month is shown in the period selector and attendance data loads for that period (or "No attendance records yet" when none). |
| AC2 | User can change the period (prev/next month or picker); data reloads for the selected period; a loading indicator is visible from request start until response. |
| AC3 | Summary/analytics block shows circular progress with attendance % and label (e.g. "85% present"); optional school target text when API provides it. |
| AC4 | Trend (this month vs last month or last 4 weeks) is shown with bar/sparkline and short text + icon; labels are accessible (not color-only). |
| AC5 | Status breakdown (Present / Absent / Other with counts) is shown as horizontal bar or chips with legend or in-bar labels; not color-only. |
| AC6 | Goal/threshold line ("On track" / "Below 90%") is shown with icon + text; meaning is clear without color (semantics). |
| AC7 | Calendar or list is visible below analytics; legend (Present, Absent, Other, No data) is present under the calendar or with the list. |
| AC8 | Tapping a day opens day drill-down (bottom sheet or inline) with daily status and optional subject/period detail when API provides; view-only. |
| AC9 | When API provides low-attendance or risk flags, an alert banner or card is shown above the calendar/list. |
| AC10 | Loading state uses skeleton/overlay and progress placeholders; no partial or fake percentages during load. |
| AC11 | Empty state shows "No attendance records yet" (or "No data for this period"); no misleading progress indicators. |
| AC12 | On API failure, an error message is shown (e.g. AppMessage) and the user can retry. |
| AC13 | All visible labels and copy are student/parent-facing (e.g. "Attendance", "Present", "Absent", "On track", "Below 90%"). |
| AC14 | Progress indicators and calendar/list are accessible: semantic labels for progress and charts, 48px min touch targets, legend and non–color-only meaning (WCAG AA). |

---

## Dependency

**SMS attendance API (daily and aggregated per student, optional subject/period, risk flags)** — Backend provides per-student attendance for a period (month/term): daily status (present/absent/other), aggregated metrics (counts, percentage), optional subject/period detail per day, optional school threshold, optional low-attendance/risk flags. **Student/active-child context** determines which student's data is shown. Flutter uses a stub/mock until the API is ready; stub should return a fixture period with per-day status and summary metrics so that loading, empty, and error states can be implemented and verified.
