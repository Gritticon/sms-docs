---
title: KYC UI Expectations
type: spec
project: kyc
last-updated: 2026-03-18
---

> ⚠️ **Pipeline sync warning** (2026-04-08): This document is marked `built` but no
> matching implementation was detected during automated codebase scan. Verify manually.

# How the KYC UI Needs to Be

Short reference for developers and designers: UI expectations for all KYC modules. Sources: `diary-ui-build-outline.md`, `timetable-ui-build-outline.md`, `kyc-overview-spec.md`, project `.cursorrules`.

---

## 1. Structure

| Layer | Pattern | Notes |
|-------|---------|--------|
| **Top** | Nav / view selector | Shell nav (e.g. Home, Transport, Diary, Account); module-specific view tabs (e.g. Day \| Week \| Month) when applicable. |
| **Below** | Date/range picker or week/month nav | **Day picker:** `[←] [Date label] [→]` — default today; tap opens calendar (OK/Cancel in a row). **Week:** prev/next week + "This week". **Month:** prev/next month + "Today". |
| **Content** | Single scrollable column | Cards or list in logical order. No extra tap to see content on open — load today/default immediately. |

- **Diary:** Day picker → period tabs (horizontal scroll) → 3 cards per period: Prerequisites/Planned → Completed → Homework/Assignment.
- **Timetable:** View toggle (Day \| Week \| Month) → day/week/month nav → content (day: period/break list; week: day cells; month: calendar grid). Selected date is shared across views; tap day in Week/Month → Day view for that date.
- **Single column** on narrow screens; optional max width (~600–720px) for main content on wide.

---

## 2. Components

**Use**

- **AppButton** for all buttons (min height 48px).
- **AppCard** (or Card with elevation 0, 12px radius, 16–24px padding) for content blocks and list rows.
- **AppDialog** for dialogs (e.g. date picker); center-aligned title and content.
- **AppMessage** for errors and notifications.
- **Field** for text inputs; **CustomSearchField** for search.
- **Theme:** `Theme.of(context).colorScheme` and `textTheme` only — no hardcoded colors or text styles.
- **Icons:** Outlined variants; theme-based colors.
- **Day picker:** "Today" when today, else formatted date (e.g. "Mar 3, 2025"); prev/next arrows; min 48px touch targets.
- **Tabs:** Horizontally scrollable when many (e.g. periods); selected state clearly distinct; min 48px height per tab.
- **Calendar:** Material or custom month grid; holiday/no-class from API shown with distinct styling.

**Avoid**

- Raw `ElevatedButton` / `OutlinedButton` / `Dialog` / `AlertDialog` / `Switch` / `TextButton`.
- Hardcoded colors (`Colors.xxx`, `Color(0xFF...)`) or shadow colors.
- Sharp corners (0px radius) unless justified.
- Filled icons unless required for branding/emphasis.

---

## 3. States

| State | When | Behavior |
|-------|------|----------|
| **Loading** | Any API request (date change, tab change, initial load, view switch) | Show a **visible** loading indicator from **request start** until response (success or error). Overlay or inline spinner; never only disable a button. |
| **Empty** | No data for current scope | Per-area friendly message (e.g. "Nothing planned for this period", "No classes for this date", "No homework for this period"). "No periods for this date" when day has no periods. |
| **Error** | API failure | Show via **AppMessage**; offer **retry**. |
| **Future date** | Diary: Completed / Homework | Placeholder or "Not available for this date" (or hide). |
| **Holiday / no-class** | Timetable | Day/Week/Month show as "Holiday" or no-class; styling distinct from normal days. |

---

## 4. Copy & Audience

- **Audience:** Student- and parent-facing only. No teacher-only wording; teachers use SMS.
- **Screen titles:** e.g. "Diary", "Timetable" — center-aligned in AppBar (`centerTitle: true`).
- **Card/section titles:** Use exact labels from specs (e.g. "Prerequisites / Planned", "Completed", "Homework / Assignment").
- **Date:** "Today" when today; otherwise platform-appropriate format (e.g. "Mon, Mar 3", "Mar 3 – 9, 2025" for week).
- **Empty states:** Short, friendly, per-context (see §3).
- **Attachments:** e.g. "Attachment" with "Open" / "Download" as appropriate.

---

## 5. Responsive & Accessibility

- **Layout:** Mobile-first; single column on narrow; same content order on wide; breakpoints via `LayoutBuilder` / `MediaQuery`.
- **Touch:** Min **48×48px** touch targets for all interactive elements.
- **Semantics:**  
  - Day picker: "Diary date" / "Timetable date", "Today" or date, "Previous day", "Next day".  
  - Period tabs: e.g. "Period 1, tab 1 of 8"; selected state announced.  
  - Cards/sections: headings for Prerequisites/Planned, Completed, Homework/Assignment; empty state text announced.  
  - Timetable: period vs break rows (break announced as "Break"); week range and day summaries; month selected date and holiday state.
- **Contrast:** WCAG AA minimum; support light and dark theme.

---

## 6. Data

- **No business logic in Flutter.** Validation, calculations, and rules live in the FastAPI backend.
- **All data from API.** KYC calls SMS APIs and only displays responses. No client-side persistence of business data beyond what’s needed for session/UI.
- **Class/section/child from context.** User does not pick class or section; inferred from logged-in student or active child (parent switcher). On child switch, reload data for the active child.
- **Attachments:** URLs or download endpoints from API; Flutter only opens/downloads.

---

## Quick checklist

- [ ] Nav/selector at top, then picker/range, then content; load today/default on open.
- [ ] AppButton, AppCard, AppDialog, AppMessage; theme colors and text styles only.
- [ ] Loading visible for every API request until response; empty and error states with clear copy and retry where applicable.
- [ ] Student/parent wording only; center-aligned dialog and AppBar titles.
- [ ] Single column on narrow; 48px touch targets; semantics for pickers, tabs, cards, and timetable views.
- [ ] No business logic in Flutter; all data from API; context-based class/section/child.
