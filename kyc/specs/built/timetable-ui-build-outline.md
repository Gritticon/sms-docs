---
title: Timetable UI Build Outline
type: spec
project: kyc
module: Timetable
last-updated: 2026-03-18
---

> **Built snapshot** — 2025-03-05 — Timetable UI build outline moved to built; Day/Week/Month views and flows documented.

# Timetable — UI Build Outline (KYC)

Outline for building the Timetable UI in KYC. Reference: **Diary** ([diary-ui-build-outline.md](diary-ui-build-outline.md)). Target user: **students** (and parents viewing child's timetable); teachers use SMS. Full requirements: [timetable-spec.md](timetable-spec.md).

---

## 1. Confirmed Assumptions


| Item              | Decision                                                                                                                 |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Primary user**  | Students (and parents viewing active child's timetable). KYC only; teachers use SMS.                                     |
| **Class/section** | User does not select; inferred from logged-in student or active child.                                                   |
| **Default view**  | **Today's** timetable in **Day view**. Load immediately on open.                                                         |
| **Day scope**     | Day picker for date (today, past, future). Same pattern as Diary: `[←] [Date] [→]` + tap to open calendar.               |
| **Periods**       | Dynamic from school (same source as diary). Day view shows ordered list of **periods and breaks**.                       |
| **Holidays**      | Backend marks holiday (and optionally no-class) days; Month view shows them; Day view shows empty state when applicable. |


---

## 2. Three Views: Day View, Week View, and Month View


| View           | Purpose                                                                                | When shown                                                                                                       |
| -------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Day view**   | Show the **timetable for the selected day**: periods, subjects, times, and **breaks**. | Default on open (today); also after selecting a date from Week or Month view or day picker.                     |
| **Week view**  | Show **one week** (e.g. Mon–Sun) with per-day summary or list; tap a day → Day view.  | User selects "Week" in view toggle; prev/next week and "This week" to navigate.                                    |
| **Month view** | **Select a date** and get an **overview of holidays** (and optionally no-class days).  | User selects "Month"; tap a day to select date, then show Day view for that date.                                |


**Switching:** User can switch between Day, Week, and Month view (e.g. tabs or toggle in AppBar). Selected date is preserved when switching.

---

## 3. Entry & Flow Summary

1. **Entry:** User opens Timetable (nav or shell).
2. **Default:** Show **Day view** for **today** — day picker + list of periods and breaks (no extra tap).
3. **Day view:**
  - **Day picker** at top: `[←] [Date label] [→]`. Same as Diary (large buttons, tap opens calendar with OK/Cancel in a row).  
  - **Content:** Chronological list of **periods** and **breaks** for the selected day (period label, subject, time; break label, time; breaks visually distinct).
4. **Week view:**
  - **Week navigation:** Prev/next week, "This week" to jump to current week. Display week range (e.g. "Mar 3 – 9, 2025").
  - **Content:** One row or card per day in the week (Mon–Sun or Sun–Sat per locale). Each day shows date, weekday name, and summary (e.g. "6 periods" or first 1–2 subjects). **Holidays** and no-class days marked. Tap a day → set selected date and show **Day view** for that date.
5. **Month view:**
  - **Month calendar** with **holidays** (and optional no-class days) marked.  
  - Prev/next month and "Today".  
  - **Tap a day** → set selected date and show **Day view** for that date (or show "View timetable" that switches to Day view).
6. **View switch:** Tabs or toggle "Day" | "Week" | "Month" (e.g. in AppBar); selected date shared between all views.

---

## 4. Screens & Components (UI Build Order)

### 4.1 Day view

- **Day picker**  
  - Reuse or mirror Diary day picker: default today, past/future, prev/next arrows, tap → calendar dialog (OK/Cancel in row). 50px height, row layout.
- **Content area (per day)**  
  - **Ordered list of slots:** each slot is either a **period** or a **break**.  
  - **Period row:** Period label (e.g. "Period 1"), **subject**, **start–end time**, optional teacher/room (if API provides). Use AppCard per row.  
  - **Break row:** Break label (e.g. "Break", "Short break", "Lunch"), **start–end time**. Visually distinct from period rows (e.g. secondary/muted card or chip).  
  - List is chronological (top to bottom = school day order).
- **Empty state:** "No classes for this date." or "Holiday" when backend indicates; or "No timetable available. Contact the school." when not configured.

### 4.2 Week view

- **Week navigation**  
  - [← Week] [Week range label, e.g. "Mar 3 – 9, 2025"] [Week →]. "This week" button to jump to the week containing today.
- **Content (per week)**  
  - One row or card per day in the displayed week (e.g. 7 days). Each day cell shows: weekday name, date, and either a short summary ("6 periods", "Holiday", "No classes") or the first one or two subjects. Days that are holidays or no-class (from API) visually distinct.  
  - **Tap a day** → set selected date and switch to **Day view** for that date.
- **Empty / no data:** If API returns no slots for a day in the week, show "No classes" or "Holiday" in that day's cell; do not hide the day.

### 4.3 Month view

- **Month calendar**  
  - One month grid (e.g. Material `CalendarDatePicker` or custom month grid).  
  - **Holiday markers:** Days that are holidays (from API) visually marked (e.g. different color, "Holiday" label, or icon).  
  - **No-class days (optional):** If API provides, show distinctly from holidays.
- **Navigation**  
  - Previous/next month; "Today" to jump to current date.  
  - Tap a day → select that date and show Day view for it (or "View timetable" for that date).
- **Empty:** If no holiday data for month, calendar still shows; only markers are absent.

### 4.4 Shared

- **View selector:** Tabs or toggle "Day" | "Week" | "Month" at top (e.g. in shell AppBar).  
- **Selected date:** Single source of truth; Day, Week, and Month views all use it.  
- **Loading:** Overlay from request start until response (same rule as Diary).  
- **Error:** AppMessage + retry.

---

## 5. States to Implement


| State                            | Where                                  | Behavior                                                                                                                     |
| -------------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Loading**                      | Day change, initial load, week change, month change | Show loading indicator from request start until response.                                                                    |
| **Empty (day)**                  | No periods for selected day            | "No classes for this date." or "Holiday" when applicable; "No timetable available. Contact the school." when not configured. |
| **Empty (week day)**            | A day in the week has no slots        | Show "No classes" or "Holiday" in that day's cell in Week view.                                                               |
| **Empty (breaks)**               | API does not return breaks             | Day view shows only period rows; no break rows.                                                                              |
| **Error**                        | API failure                            | AppMessage + retry.                                                                                                          |
| **Month view – no holiday data** | Month calendar                         | Calendar shown; holidays simply not marked.                                                                                  |
| **Break row**                    | Day view list                          | Show break label and time; styling clearly different from period rows.                                                       |


---

## 6. Copy & Labels (student-facing)

- Screen title: **Timetable**.  
- View tabs: **Day** | **Week** | **Month**.  
- Day picker: "Today" or formatted date (e.g. "Mon, Mar 3").  
- Period row: period label, subject name, time range; optional "Teacher: …", "Room: …".  
- Break row: "Break", "Short break", "Lunch" (or labels from API) + time range.  
- Empty: "No classes for this date." / "Holiday" / "No timetable available. Contact the school."  
- Month view: "Today" for current date; holiday indicator or label on calendar cells.

---

## 7. Backend Dependencies (high level)

- **Timetable API (day):** For active student and a given date, return ordered list of **slots**. Each slot is either:  
  - **Period:** period label, subject, start/end time, optional teacher, room.  
  - **Break:** label, start/end time.  
  KYC renders in order; no business logic in Flutter.
- **Timetable API (week):** For active student and a date range (e.g. start and end of week), return slots per day, or list of dates with slots. Enables Week view without N day-API calls. Alternatively, Week view can call the day API once per day in the week (max 7).
- **Timetable API (month / holidays):** For a month or date range, return which dates are **holidays** (and optionally "no classes"). Flutter only displays and handles tap-to-select.
- **Multi-child:** When parent switches child, timetable reloads for active child.

Details (endpoints, payloads, errors) belong in SMS/SMS-API docs; KYC consumes and displays only.

---

## 8. UX Guidance (for Flutter implementation)

**Day view structure (top to bottom):**

1. **View toggle:** "Day" | "Week" | "Month" (tabs or segmented control, e.g. in AppBar).
2. **Day picker row:** `[←] [Date] [→]` — same as Diary (50px height, FilledButton.tonal, calendar dialog with OK/Cancel in row).
3. **Content:** Single scrollable list of period cards and break rows in time order.

**Month view structure:**

1. **View toggle:** Same as above.
2. **Month navigation:** Prev/next month, "Today".
3. **Calendar grid:** Month with holiday (and optional no-class) markers; tap day → set date and show Day view.

**Week view structure:**

1. **View toggle:** Same as above (Day | Week | Month).
2. **Week navigation:** Prev/next week, "This week"; show week range (e.g. "Mar 3 – 9, 2025").
3. **Week content:** One row or card per day; weekday, date, summary or first subjects; holiday/no-class marked; tap day → set date and show Day view.

**Component UX:**

- Day picker: Reuse Diary day picker pattern; min 48px touch targets.  
- Period row: AppCard; period label, subject, time prominent; teacher/room secondary.  
- Break row: Visually distinct (e.g. FilledButton.tonal or surfaceContainerHighest); break label + time.  
- Responsive: Single column on narrow; same order on wide; optional max width for content.

**Loading and empty states:**

- One loading overlay from request start until response.  
- Empty states as in §5.

**Accessibility:**

- Day picker: semantics for "Timetable date", "Today"/date, "Previous day", "Next day".  
- Period/break rows: section or list semantics; break rows announced as "Break" not "Period".  
- Month calendar: selected date and holiday state announced.  
- Week view: week range and each day's summary announced; tap day announced as "View timetable for [date]".

---

## 9. Reference

- Full requirements, user stories, MVP scope: **timetable-spec.md** (this folder).  
- Diary UI pattern (day picker, loading, error): **diary-ui-build-outline.md** (this folder).  
- KYC specs lifecycle: **../planned/specs-lifecycle-process.md**.  
- PM acceptance criteria and dependency: **timetable-ui-pm-addendum.md** (this folder).

