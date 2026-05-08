---
title: Diary UI Build Outline
type: spec
project: kyc
module: Diary
last-updated: 2026-03-18
---

> **Built snapshot** — Diary UI implemented (day picker, period tabs, 3 cards). API integration may be pending.

# Diary — UI Build Outline (KYC)

Outline for building the Diary UI in KYC. Target user: **students only** (teachers use SMS). For full requirements and user stories, see `arch/diary-ui-specification.md`.

---

## 1. Confirmed Assumptions

| Item | Decision |
|------|----------|
| **Primary user** | Students only. KYC is the student app; teachers use SMS. Wording and flows are student-centric. |
| **Future dates** | User can pick a future date. For future dates: show only **Prerequisites/Planned** card; show placeholders or empty state for **Completed** and **Homework/Assignment**. |
| **Prerequisites/Planned** | Same concept: one card combining (a) what you should know before this period, (b) what is planned to be taught. Single card label, e.g. **Prerequisites / Planned**. |
| **Periods per day** | Dynamic from school settings. UI must support variable count (e.g. 6–10+ periods) via scrollable tabs or equivalent. |
| **Class/section** | User does not select class or section; inferred from logged-in student context. |
| **Previous diary** | Browsing previous diaries = picking a **past date** in the same day picker (no separate "diary history" screen). |

---

## 2. Entry & Flow Summary

1. **Entry:** User opens Diary (e.g. from KYC nav).
2. **Default:** Show **today's** date and load that day's periods as tabs immediately (no extra tap).
3. **Day picker:** User can change date (past or future). On date change → reload periods and content for that date.
4. **Period tabs:** One tab per period for the selected day. Tabs are dynamic (from school settings). Use horizontally **scrollable** tab row when there are many periods.
5. **Per-tab content:** Each tab shows **3 cards** in order:
   - **Prerequisites / Planned** — What to know before + what's planned for this period.
   - **Completed** — What was covered/done in this period.
   - **Homework / Assignment** — Assignments and class notes; may include attachments.
6. **Previous diaries:** Same screen; user picks a past date in the day picker to view that day's periods and cards.
7. **Optional UX:** Previous/next day controls (arrows or swipe) for quick move to yesterday/tomorrow without opening full calendar.

---

## 3. Screens & Components (UI Build Order)

### 3.1 Diary screen (main)

- **Day picker**  
  - Default: today.  
  - Allows past and future dates.  
  - On change: trigger load of periods and content for selected date.

- **Period tabs**  
  - One tab per period (labels from API, e.g. "Period 1", "Period 2", or subject name if provided).  
  - Horizontally scrollable (dynamic count).  
  - Selected tab drives which period's cards are shown.

- **Content area (per period)**  
  - Three cards stacked vertically (or scrollable column):
    1. **Prerequisites / Planned**
    2. **Completed**
    3. **Homework / Assignment** (with class notes and assignment details; attachment indicators and open/download)

### 3.2 Cards (reusable)

- **Card: Prerequisites / Planned**  
  - Single card. Content from API (text/structured). Empty state: "Nothing planned for this period."

- **Card: Completed**  
  - What was covered in this period. Empty state: "Nothing completed yet" (or hide if future date).

- **Card: Homework / Assignment**  
  - List or block of homework items; class notes; assignment details.  
  - Attachment support: show indicator per item; tap to open/download.  
  - Empty state: "No homework for this period."

### 3.3 Optional / later

- **Previous/next day** — Arrows or swipe to adjacent day without opening calendar.
- **Homework detail** — If list is summary-only, a detail screen for one homework item (description, due date, attachments).

---

## 4. States to Implement

| State | Where | Behavior |
|-------|--------|----------|
| **Loading** | Day change, tab change, card content | Show loading indicator from request start until response (success or error). Per project rules: loading visible for every API request. |
| **Empty** | Each card | Prerequisites/Planned: "Nothing planned for this period." Completed: "Nothing completed yet" (or hide for future). Homework: "No homework for this period." |
| **No periods for date** | Main content | e.g. "No periods for this date." |
| **Error** | Any API failure | Show error (e.g. AppMessage); allow retry. |
| **Future date** | Completed & Homework cards | Show placeholder/empty or "Not available for this date." |

---

## 5. Copy & Labels (student-facing)

- Screen title: e.g. **Diary**.
- Card titles: **Prerequisites / Planned**, **Completed**, **Homework / Assignment**.
- Day picker: use platform-appropriate date label (e.g. "Today", "Mar 3, 2025").
- Period tabs: use labels from API (e.g. "Period 1", "Period 2", or subject/slot name).
- Attachments: e.g. "Attachment" or icon + "Open" / "Download" depending on platform and API.

---

## 6. Backend Dependencies (high level)

- **Per-date periods:** API returns list of periods for a given date (for current student context).
- **Per-period content:** API returns for each period: prerequisites/planned, completed, homework (with optional attachments).
- **Future dates:** API supports future date; returns at least Prerequisites/Planned; Completed/Homework may be empty or omitted.
- **Attachments:** URLs or download endpoints from API; Flutter only opens/downloads, no business logic.

Details (endpoints, payloads, errors) belong in SMS/SMS-API docs; KYC consumes responses and handles loading/empty/error in UI.

---

## 8. UX guidance (for Flutter implementation)

**Screen structure (top to bottom):**

1. **Day picker row:** `[←] [Date label] [→]`. Date label = "Today" when today, else "Mar 3, 2025". Tap date opens date picker; arrows = prev/next day.
2. **Period tabs:** Horizontal scroll (e.g. SingleChildScrollView + row of tabs). Min height 48px per tab; selected tab visually distinct (e.g. filled background or indicator).
3. **Content area:** Three cards in a column: Prerequisites / Planned → Completed → Homework / Assignment. Use AppCard for each.

**Component UX:**

- **Day picker:** "Today" vs formatted date; prev/next arrows beside date; min 48px touch targets.
- **Cards:** AppCard for all three; empty copy exactly as in §4; future date: Completed/Homework "Not available for this date" or hide.
- **Responsive:** Narrow = single column, full width; wide = same order, optional max width (~600–720px) for card column.

**Loading and empty states:**

- One loading overlay (full content area or full screen) from request start until response; no request without visible loading.
- Empty states per card as in §4; "No periods for this date" in content area when no periods.

**Accessibility:**

- Day picker: semantics for "Diary date", "Today"/date, "Previous day", "Next day".
- Period tabs: each tab labeled (e.g. "Period 1, tab 1 of 8"), selected state announced.
- Cards: section headings (Prerequisites/Planned, Completed, Homework/Assignment); empty state text announced.

---

## 9. Reference

- Full requirements, user stories, edge cases, MVP scope: **`arch/diary-ui-specification.md`** (update that doc with "day picker + period tabs + 3 cards" and KYC-only student context when revising arch).
- KYC specs lifecycle: **`kyc/docs/planned/specs-lifecycle-process.md`**.
- PM acceptance criteria and dependency: **`diary-ui-pm-addendum.md`** (this folder).
- Previous built snapshot (older scope): **`diary-spec.md`** (this folder).
