---
title: Attendance UI Build Outline
type: spec
project: kyc
module: Attendance
last-updated: 2026-03-18
---

> **Built snapshot** — Attendance UI implemented (period selector, analytics block, calendar/list, day drill-down, alerts). Stub data; API integration may be pending.

# Attendance — UI Build Outline (KYC)

Outline for building the Attendance UI in KYC. Target user: **students and parents** (view-only; data for active student/selected child). Full requirements: [attendance-analytics-spec.md](attendance-analytics-spec.md). Detailed UX and progress indicators: [attendance-ui-ux-design.md](attendance-ui-ux-design.md).

---

## 1. Confirmed Assumptions

| Item | Decision |
|------|----------|
| **Primary user** | Students and parents (view-only). Data for active student or selected child; teachers use SMS. |
| **Class/section** | User does not select; inferred from logged-in student or active child context. |
| **Default period** | Current month. Load immediately on open. |
| **View-only** | No editing, no leave requests, no justification flows in KYC. |
| **Data source** | All metrics, counts, percentages, and risk flags from SMS API; Flutter displays only. |
| **Period scope** | Month or term; user navigates via prev/next and optional picker. |

---

## 2. Entry & Flow Summary

1. **Entry:** User opens Attendance (e.g. from KYC nav).
2. **Default:** Show **current month**; load attendance data for that period (summary + calendar/list).
3. **Period selector:** `[←] [Month label / term] [→]` at top. Tap opens month/period picker. On change → reload data; loading visible from request start until response.
4. **Summary/analytics block:** Below selector. Circular progress (attendance %), trend (this vs last month or last 4 weeks), status breakdown (Present / Absent / Other), goal line ("On track" / "Below 90%"). User sees "how am I doing?" before raw calendar.
5. **Alerts:** When API provides low-attendance or risk flags, show compact banner or card above calendar/list.
6. **Calendar or list:** Below analytics (and alerts if any). Month grid or list of days with status; legend under (Present, Absent, Other, No data). Tap a day → day drill-down.
7. **Day drill-down:** Bottom sheet or inline expansion: daily status, optional subject/period breakdown and remarks if API provides; view-only.

---

## 3. Screens & Components (UI Build Order)

### 3.1 Period selector

- **Pattern:** Same as Diary/Timetable month nav: `[←] [Month / term label] [→]`. Tap label opens month or period picker (e.g. calendar or dropdown). Prev/next steps one month (or term).
- **Default:** Current month.
- **Min 48px touch targets** for arrows and label.
- **On change:** Trigger load for selected period; show loading from request start until response.

### 3.2 Summary / analytics block

- **Placement:** Directly below period selector. Single AppCard or grouped cards.
- **Content (see [attendance-ui-ux-design.md](attendance-ui-ux-design.md) §2 for detail):**
  - **Circular progress** — Overall attendance % (e.g. 85% present). Label: "X% present" or "Attendance: X%". Optional: "School target: 90%" when API provides threshold. Use theme colors; 80–120px diameter on mobile.
  - **Trend** — "This month vs last month" or "Last 4 weeks" with compact bar or sparkline; short text + icon (e.g. "Up from last month"). Accessible labels for bars (e.g. "Week 1", "Week 2").
  - **Status breakdown** — Horizontal stacked bar or chips: Present / Absent / Other with counts (e.g. "Present 17", "Absent 2", "Other 1"). Legend or in-bar labels; not color-only.
  - **Goal / threshold** — One line: "On track" or "Below 90%" with icon + text; use success/warning theme colors; semantics so meaning is clear without color.
- **Loading:** Skeleton placeholders for progress, bars, and numbers; no partial or stale percentages.
- **Empty:** "No data for this period" or hide progress; no circular progress with 0% unless product explicitly wants it.

### 3.3 Alerts

- **When:** API provides low-attendance or risk flags for the student.
- **Placement:** Compact banner or card **above** the calendar/list (not inside it).
- **Content:** Short message (e.g. "Attendance below school target"); optional link to detail. Use theme error/warning container; icon + text.

### 3.4 Calendar or list

- **Placement:** Below analytics block (and alerts if any). Single scrollable column; no Summary vs Calendar tabs for MVP.
- **Calendar:** Month grid; each day cell shows status (present/absent/other) with distinct icon or pattern + color. **Legend** under calendar: "Present", "Absent", "Other", "No data".
- **List (alternative):** One row per day: date (e.g. "Mon, Mar 3"), status label, optional icon. Same legend or column header.
- **Interaction:** Tap day → day drill-down. Min 48px touch target for cells or rows.

### 3.5 Day drill-down

- **Trigger:** Tap a day in calendar or list.
- **Presentation:** Bottom sheet or inline expansion.
- **Content:** Overall status for that day; optional subject-wise or period-wise breakdown and remarks if API provides (F5). Otherwise simplified view: overall status + any notes. View-only; no editing.

---

## 4. States to Implement

| State | Where | Behavior |
|-------|--------|----------|
| **Loading** | Period change, initial load | Skeleton or overlay for content block; progress indicators as placeholders. Do not show partial or stale percentages. Period selector can stay interactive. |
| **Empty** | No attendance records for selected period | "No attendance records yet" in summary/analytics area. No misleading progress (e.g. no 0% ring unless product wants it). Optional short copy above calendar. |
| **Error** | API failure | AppMessage + Retry. Keep period selector visible; content shows error state. Do not show progress with fake data. |

---

## 5. Copy & Labels (student/parent-facing)

- Screen title: **Attendance**.
- Period selector: month name (e.g. "March 2025") or term label.
- Progress: "X% present", "Attendance: X%"; optional "School target: 90%".
- Trend: "Up from last month", "Same as last month", "Down from last month"; or "Last 4 weeks" with week labels.
- Breakdown: "Present", "Absent", "Other" with counts.
- Goal: "On track", "Below 90%" (or school threshold text from API).
- Legend: "Present", "Absent", "Other", "No data".
- Empty: "No attendance records yet" / "No data for this period".
- Alerts: Short, non-technical message (e.g. "Attendance below school target").

---

## 6. Backend Dependencies (high level)

- **SMS attendance API** — Per-student, per period (month/term): daily status (present/absent/other), aggregated metrics (counts, percentage), optional subject/period detail per day, optional school threshold, optional risk/low-attendance flags. KYC displays only; no business logic in Flutter.
- **Active-child context** — When parent switches child, attendance reloads for selected student.
- **Calendar/date** — KYC uses same date-range and calendar patterns as Diary/Timetable; backend defines working days and period boundaries.

Details (endpoints, payloads, errors) belong in SMS/SMS-API docs.

---

## 7. UX / Accessibility

- **Focus order:** Period selector → summary/analytics (progress, then trend, then breakdown, then goal) → alerts → calendar/list → day detail. Logical and predictable.
- **Progress and charts:** Circular progress: `semanticLabel` e.g. "Attendance 85 percent". Bars/sparklines: describe in text or expose key values for screen readers. Status breakdown: "Present 17", "Absent 2", "Other 1" announced; not color-only.
- **Goal/threshold:** "On track" / "Below 90%" announced as text; icon has `semanticLabel`.
- **Calendar/list:** Day cells/rows have labels (e.g. "March 3, Present"); legend items focusable or clearly associated. Sufficient contrast (theme colors; WCAG AA).
- **Touch:** Min 48×48px for picker arrows, day cells, list rows, retry.
- **Loading/empty/error:** Loading announced (e.g. "Loading attendance"); empty and error as clear, short messages.

See [attendance-ui-ux-design.md](attendance-ui-ux-design.md) and [kyc-ui-expectations.md](kyc-ui-expectations.md) for full guidance.

---

## 8. Reference

- Full requirements, user stories, MVP scope: **attendance-analytics-spec.md** (this folder).
- UX design (progress indicators, states, a11y): **attendance-ui-ux-design.md** (this folder).
- KYC specs lifecycle: **specs-lifecycle-process.md** (planned folder).
- PM acceptance criteria and dependency: **attendance-ui-pm-addendum.md** (this folder).
