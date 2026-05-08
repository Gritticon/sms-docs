---
title: Marks UI Build Outline
type: spec
project: kyc
module: Marks
last-updated: 2026-03-18
---

> **Built snapshot** — Marks UI implemented (term selector, exam list, exam detail with subject-wise marks). Stub data; API integration may be pending.

# Marks (Markbooks) — UI Build Outline (KYC)

Outline for building the Marks UI in KYC. Target user: **students and parents** (view-only; data for active student/selected child). Full requirements: [marks-markbooks-spec.md](marks-markbooks-spec.md). UX design: [marks-ui-ux-design.md](marks-ui-ux-design.md). PM acceptance criteria: [marks-ui-pm-addendum.md](marks-ui-pm-addendum.md).

---

## 1. Confirmed Assumptions

| Item | Decision |
|------|----------|
| **Primary user** | Students and parents (view-only). Data for active student or selected child; teachers use SMS. |
| **Default period** | Current term (or first available term from API). Load exam list on open. |
| **View-only** | No editing; all calculations and grades from backend. |
| **Data source** | Exam list, overall indicator per exam, subject-wise marks, and optional flags from SMS API; Flutter displays only. |
| **Drill-down** | Tap exam → exam detail (new route or equivalent): overall + subject list. Back returns to list. |

---

## 2. Entry & Flow Summary

1. **Entry:** User opens Marks (e.g. from KYC nav).
2. **Default:** Show **current term** (or first available); load exam list for that term.
3. **Period/term selector:** `[←] [Term label] [→]` at top. Tap label opens term picker. On change → reload list; loading visible from request start until response.
4. **Optional summary (P1):** If API provides trend/summary, show one AppCard with bar chart or compact list of exams with overall %; accessible labels. Place below selector, above exam list.
5. **Exam list:** Below selector (and summary if present). One row/card per exam: name, overall indicator (e.g. 82% or Grade A), or "Marks not yet published". Tap row → exam detail.
6. **Exam detail:** Overall indicator (e.g. "Overall: 82%") + subject-wise list (subject, obtained, max, grade, remark). "Marks not yet published" when exam has no published marks; "Pending" for unreleased subjects. Optional: concern banner when backend flags (P1).
7. **Back:** From detail, back returns to exam list (term unchanged).

---

## 3. Screens & Components (UI Build Order)

### 3.1 Period/term selector

- **Pattern:** Same as Diary/Timetable: `[←] [Term label] [→]`. Tap label opens term picker (e.g. dropdown or dialog). Prev/next step one term.
- **Default:** Current term or first available from API.
- **Min 48px touch targets** for arrows and label.
- **On change:** Trigger load for selected term; show loading from request start until response.

### 3.2 Optional summary block (P1)

- **When:** API provides trend/summary data.
- **Placement:** Directly below term selector. Single AppCard.
- **Content:** Simple bar chart or list of exams with overall %; axes or bars labeled; not color-only. See [marks-ui-ux-design.md](marks-ui-ux-design.md) §Progress visualization.
- **Loading:** Skeleton placeholder when loading summary; omit block if API does not provide.

### 3.3 Exam list

- **Placement:** Below term selector (and summary if present). Single scrollable column.
- **Content:** One AppCard or list row per exam: exam name, overall indicator (e.g. "82%" or "Grade A"), or "Marks not yet published". Min 48px tap target per row.
- **Interaction:** Tap → navigate to exam detail (pass exam id/context); load detail data; show loading until response.
- **Loading:** Skeleton or overlay for list from request start until response.
- **Empty:** "No exams for this term" or "No marks available yet".

### 3.4 Exam detail

- **Entry:** From exam list tap. Route or bottom sheet; recommend route for back stack and focus.
- **Header:** AppBar with exam name (or "Exam details"), back.
- **Content:** (1) Overall indicator line/card (e.g. "Overall: 82%" or total/max + grade). (2) Subject-wise list: subject name, marks obtained, max, grade, remark; "Pending" where applicable. (3) If backend flags concerns (P1): compact banner or inline with icon + text.
- **Loading:** Loading for detail content from request start until response.
- **Empty (unpublished):** "Marks not yet published" (no subject table or fake data).
- **Error:** AppMessage + Retry.

---

## 4. States to Implement

| State | Where | Behavior |
|-------|--------|----------|
| **Loading** | List (open, term change); detail (tap exam) | Skeleton or overlay from request start until response. No partial or fake percentages. |
| **Empty** | No exams for term; exam has no published marks | List: "No exams for this term" / "No marks available yet". Detail: "Marks not yet published". |
| **Error** | API failure | AppMessage + Retry. Keep selector visible; do not show marks with fake data. |

---

## 5. Copy & Labels (student/parent-facing)

- Screen title: **Marks** or **Marks & Markbooks**.
- Selector: Term label (e.g. "Term 1", "Semester 1").
- List: Exam name; "Overall: 82%" or "Grade A"; "Marks not yet published".
- Detail: "Overall"; "Subject", "Marks", "Out of" / "Max", "Grade", "Remark"; "Marks not yet published"; "Pending".
- Concern: Short text + icon (e.g. "Below passing in Mathematics").
- Empty: "No exams for this term", "No marks available yet".
- Error: Short message + "Retry".

---

## 6. Backend Dependencies (high level)

- **SMS marks API** — Exam list per term (with optional overall indicator per exam); subject-wise marks for an exam (obtained, max, grade, remark); unpublished/pending flags; optional summary/trend (P1); optional concern flags (P1). KYC displays only; no business logic in Flutter.
- **Active-child context** — On parent child switch, marks reload for selected student.
- **Term configuration** — Backend defines terms and ordering; KYC uses same for selector.

Details (endpoints, payloads, errors) belong in SMS/API docs.

---

## 7. UX / Accessibility

- **Focus order:** Term selector → optional summary → exam list (per row) → detail: overall → subject list (per row) → concern banner if any.
- **Semantics:** Selector, exam rows, overall indicator, and any chart have semantic labels; grades/concerns not color-only.
- **Touch:** Min 48×48px for selector, list rows, retry, and detail actions.
- **Loading/empty/error:** Loading announced (e.g. "Loading marks"); empty and error as clear, short messages.

See [marks-ui-ux-design.md](marks-ui-ux-design.md) and [kyc-ui-expectations.md](kyc-ui-expectations.md) for full guidance.

---

## 8. Reference

- Full requirements, user stories, MVP scope: **marks-markbooks-spec.md** (this folder).
- UX design (structure, flows, components, states, a11y): **marks-ui-ux-design.md** (this folder).
- PM acceptance criteria and dependency: **marks-ui-pm-addendum.md** (this folder).
- KYC specs lifecycle: **specs-lifecycle-process.md** (planned folder).
