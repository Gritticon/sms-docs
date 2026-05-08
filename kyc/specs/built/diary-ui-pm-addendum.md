---
title: Diary UI PM Addendum
type: spec
project: kyc
module: Diary
last-updated: 2026-03-18
---

> **Built snapshot** — Diary UI implemented. See **diary-ui-build-outline.md** in this folder.

# Diary UI — PM Addendum (Flutter MVP)

Traceable acceptance criteria and dependency for the KYC Diary screen. Source: [diary-ui-build-outline.md](diary-ui-build-outline.md).

---

## Acceptance criteria (checklist)

| ID | Criterion |
|----|-----------|
| AC1 | On open Diary, today's date is shown in the day picker and period tabs load for that date (or "No periods for this date" when none). |
| AC2 | User can change the date (past or future); periods and card content reload for the selected date; a loading indicator is visible from request start until response. |
| AC3 | Period tabs are horizontally scrollable when there are many periods; selecting a tab shows that period's three cards (Prerequisites/Planned, Completed, Homework/Assignment) in order. |
| AC4 | Each card shows the correct empty-state message when the API returns no content; for a future date, Completed (and Homework if applicable) show placeholder/empty or "Not available for this date." |
| AC5 | On API failure, an error message is shown (e.g. AppMessage) and the user can retry. |
| AC6 | All visible labels and copy are student-facing (e.g. "Diary", "Prerequisites / Planned", "Completed", "Homework / Assignment"). |
| AC7 | Homework/Assignment card shows attachment indicators where applicable; user can tap to open or download (behavior depends on API/stub). |

---

## Dependency

**Diary API (periods by date, per-period content)** — Backend. Flutter uses a stub/mock until the API is ready. The stub should return periods for a given date and, per period, prerequisites/planned, completed, and homework (with optional attachment metadata) so that loading, empty, and error states can be implemented and verified.
