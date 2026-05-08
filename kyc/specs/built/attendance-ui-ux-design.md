---
title: Attendance UI UX Design
type: spec
project: kyc
module: Attendance
last-updated: 2026-03-18
---

> **Built snapshot** — Attendance UI implemented. See **attendance-ui-build-outline.md** in this folder.

# Attendance UI — UX Design Note

Design-time UX guidance for the KYC Attendance & Analytics screen. Audience: **students and parents** (view-only; data for active student/selected child). Aligns with [attendance-analytics-spec.md](attendance-analytics-spec.md) and KYC UI expectations (day/month picker, AppCard, theme colors, 48px touch targets, loading from request start until response).

---

## 1. Structure

**Order of blocks (top to bottom) — analytics-first, scannable:**

1. **Period selector** — Month/term picker at top (same pattern as Diary/Timetable: `[←] [Month label / term] [→]`; tap opens month/period picker). Default: current month. Min 48px touch targets.
2. **Summary / analytics block** — Key metrics and progress indicators (see §2). Single AppCard or grouped cards so the user sees "how am I doing?" before raw data.
3. **Calendar or list view** — Day-by-day status (calendar grid or list). Placed below analytics so users scan progress first, then drill into dates.
4. **Alerts** — Low-attendance or threshold alerts (if API provides flags). Show as a compact banner or card **above** the calendar/list when present; do not hide inside the calendar.
5. **Day drill-down** — Tap a day opens detail (e.g. bottom sheet or inline expansion): daily status, optional subject/period breakdown, remarks. No editing; view-only.

**Rationale:** Metrics visible before calendar reduces cognitive load and answers "Am I on track?" immediately. Period selector first keeps the pattern consistent with Diary/Timetable.

**Optional:** On wide layouts, analytics can sit in a left column or stay stacked; keep the same order (selector → analytics → calendar → alerts → detail).

---

## 2. Analytics & progress indicators

All indicators must be **accessible** (not color-only): pair color with text, icons, or pattern. Student/parent-friendly labels; work on mobile and desktop.

### 2.1 Attendance percentage

- **Primary control:** **Circular progress** for overall attendance % (e.g. 85% present). Large enough to read at a glance (e.g. 80–120px diameter on mobile).
- **Label:** Always show numeric value and word: e.g. "85% present" or "Attendance: 85%". Use `Theme.of(context).textTheme` and `colorScheme`.
- **Optional comparison:** If API provides a school threshold (e.g. 90%), show a subtle reference: e.g. "School target: 90%" or a small tick/mark on the ring. Do not rely on color alone to convey above/below target.

### 2.2 Trend or period comparison

- **Pattern:** "This month vs last month" or "Last 4 weeks" with a **compact bar or sparkline** (e.g. 4 short bars, one per week). Label axes or bars (e.g. "Week 1", "Week 2") so it's not color-only.
- **Copy:** Short text such as "Up from last month" or "Same as last month" with an icon (e.g. trend up/down/neutral). Ensure icon has `semanticLabel` for screen readers.

### 2.3 Status breakdown

- **Pattern:** **Horizontal stacked bar** or **small chips** for Present / Absent / Other counts or ratio.
- **Bar:** Single bar with segments (e.g. green = present, amber = absent, gray = other); each segment labeled (e.g. "Present 17", "Absent 2", "Other 1") in or under the bar. Legend under the bar if needed.
- **Chips:** Alternatively, three chips with count + label (e.g. "Present 17", "Absent 2", "Other 1"). Use theme colors and ensure sufficient contrast and focus states.

### 2.4 Goal or threshold

- **Pattern:** One-line status with **icon + short text + color**. E.g. "On track" (check icon, success color), "Below 90%" (warning icon, warning color). Always pair with text so meaning is clear without color.
- **Placement:** Directly under or beside the circular progress. Accessible: announce "On track" or "Below 90%" via semantics; do not rely on icon color alone.

**Where progress indicators sit when loading/empty:** See §4.

---

## 3. Calendar / list

- **Placement:** Below the analytics block. No tabs required for MVP; single scrollable column: period selector → analytics card(s) → alerts (if any) → calendar or list.
- **Alternative (if needed later):** Tabs "Summary" vs "Calendar" — Summary = analytics + status breakdown; Calendar = same analytics compact at top + calendar below. Prefer one scroll to avoid extra tap.
- **Calendar:** Month grid; each day cell shows status (present/absent/other) with **distinct icon or pattern + color**. Include a **legend** under the calendar (e.g. "Present", "Absent", "Other", "No data") so meaning is not color-only.
- **List:** If list view is used, one row per day: date (e.g. "Mon, Mar 3"), status label, optional icon. Same legend or column header so status is clear.
- **Interaction:** Tap day → day drill-down (detail). Use 48px min touch target for day cells or list rows.

---

## 4. States

### Loading

- **Scope:** Whole block (period selector can stay interactive for quick switch; content area shows loading). Show **skeleton or overlay** from request start until response (per project rules).
- **Progress indicators:** Show skeleton placeholders for circular progress, bars, and summary numbers (e.g. gray blocks matching layout). Do not show partial or stale percentages during load.

### Empty

- **When:** No attendance records for the selected period (e.g. "No attendance records yet").
- **Progress indicators:** Hide or replace with neutral state: e.g. "No data for this period" in the analytics area; no circular progress with 0% unless product explicitly wants that. Prefer one clear empty message in the summary block and optional short copy above calendar.

### Error

- **Pattern:** Message (e.g. AppMessage) + **Retry** control. Keep period selector visible; content area shows error state. Do not show progress indicators with fake data; show error and retry only.

---

## 5. Accessibility

- **Focus order:** Period selector → summary/analytics (progress, then trend, then breakdown, then goal) → alerts → calendar/list → day detail. Logical and predictable.
- **Progress and charts:**
  - Circular progress: `semanticLabel` e.g. "Attendance 85 percent".
  - Bars/sparklines: describe in text (e.g. "Last 4 weeks: 18, 20, 19, 20 days present") or expose key values so screen reader users get the same info.
  - Status breakdown: ensure "Present 17", "Absent 2", "Other 1" are announced; not color-only.
- **Goal/threshold:** "On track" / "Below 90%" announced as text; icon has `semanticLabel`.
- **Calendar:** Day cells have labels (e.g. "March 3, Present"); legend items are focusable or clearly associated. Sufficient contrast for present/absent/other (theme colors; WCAG AA).
- **Touch:** Min 48×48px for all tappable elements (picker arrows, day cells, list rows, retry).
- **Loading/empty/error:** Loading state announced (e.g. "Loading attendance"); empty and error states read as clear, short messages.

---

## Summary checklist (for implementation)

- [x] Structure: Period selector → Analytics (progress indicators) → Alerts (if any) → Calendar/list → Day drill-down.
- [x] Circular progress for overall %; label "X% present"; optional school target text.
- [x] Trend: "This month vs last month" or last 4 weeks with bar/sparkline + short text + icon; accessible labels.
- [x] Status breakdown: horizontal bar or chips (Present / Absent / Other) with counts; legend or in-bar labels.
- [x] Goal: "On track" / "Below 90%" with icon + text; not color-only.
- [x] Calendar below analytics; legend under calendar; tap day → detail.
- [x] Loading: skeleton/overlay for whole block; progress indicators as placeholders.
- [x] Empty: "No attendance records yet"; no misleading progress.
- [x] Error: message + retry.
- [x] Accessibility: focus order, semantic labels for progress/charts, 48px targets, contrast, non–color-only meaning.

This structure is testable against the acceptance criteria in [attendance-ui-pm-addendum.md](attendance-ui-pm-addendum.md).
