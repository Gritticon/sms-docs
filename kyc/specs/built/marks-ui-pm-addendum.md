---
title: Marks UI PM Addendum
type: spec
project: kyc
module: Marks
last-updated: 2026-03-18
---

> **Built snapshot** — Marks UI implemented. See **marks-ui-build-outline.md** in this folder.

# Marks (Markbooks) UI — PM Addendum (Flutter MVP)

Traceable acceptance criteria and dependency for the KYC Marks screen. Source: [marks-markbooks-spec.md](marks-markbooks-spec.md), [marks-ui-ux-design.md](marks-ui-ux-design.md), [marks-ui-build-outline.md](marks-ui-build-outline.md).

---

## Scope (MVP)

| Item | Definition |
|------|------------|
| **Data** | Exams (list per term), per-exam overall indicator (e.g. percentage or grade), subject-wise marks (obtained, max, grade, remark). Optional: trend/summary when API provides (P1). |
| **Default view** | Marks list for **current term** (or first available term from API). Period/term selector at top; exam list below. |
| **Drill-down** | Tap exam → **Exam detail** screen: overall indicator + subject-wise list (marks obtained, max, grade, remark). "Marks not yet published" when exam has no published marks; "Pending" for individual subjects when applicable. |
| **Users** | Students and parents (view-only). Data for active student or selected child; on child switch, marks reload for new child. |
| **Dependencies** | SMS marks API (exam list, subject-wise marks, overall indicator, optional flags); academic term configuration; active-child context. |

---

## Acceptance criteria (checklist)

| ID | Criterion |
|----|-----------|
| AC1 | On open Marks, the current term (or first available) is shown in the period selector and the exam list loads for that term (or "No exams for this term" when none). |
| AC2 | User can change the term (prev/next or picker); exam list reloads for the selected term; a loading indicator is visible from request start until response. |
| AC3 | Exam list shows one row per exam with exam name and overall indicator (e.g. percentage or grade) from the backend; each row has min 48px touch target and is tappable. |
| AC4 | When an exam has no published marks, the list shows "Marks not yet published" (or equivalent) for that exam; tapping it opens detail with the same message (no empty or misleading data). |
| AC5 | Tapping an exam opens the exam detail view: overall indicator (e.g. "Overall: 82%" or total/max and grade) and subject-wise list (subject, marks obtained, max, grade, remark). |
| AC6 | Subject rows show "Pending" or equivalent when a subject's marks are not yet released; overall indicator reflects only published data per backend. |
| AC7 | When backend flags subjects or exams as areas of concern (e.g. failing), the UI shows a clear label and icon (not color-only) in list or detail as defined in UX. |
| AC8 | When parent switches active child, Marks data reloads and shows the selected child's exams and marks. |
| AC9 | Optional (P1): When API provides trend/summary, a summary block (e.g. bar chart or list of exams with overall %) is shown above the exam list with accessible labels. |
| AC10 | Loading state uses skeleton or overlay from request start until response; no partial or fake percentages during load. |
| AC11 | Empty state shows "No exams for this term" or "No marks available yet" as appropriate; exam detail shows "Marks not yet published" when applicable. |
| AC12 | On API failure, an error message is shown (e.g. AppMessage) and the user can retry. |
| AC13 | All visible labels and copy are student/parent-facing (e.g. "Marks", "Overall", "Subject", "Marks not yet published", "Pending"). |
| AC14 | Focus order, semantic labels for list and any chart, min 48px touch targets, and non–color-only meaning for grades and flags (WCAG AA). |

---

## Dependency

**SMS marks API** — Backend provides: (1) exam list per term (with optional overall indicator per exam), (2) subject-wise marks for a selected exam (obtained, max, grade, remark), (3) clear indication of unpublished exams or pending subjects, (4) optional overall/trend data for summary (P1), (5) optional flags for areas of concern. **Active-child/student context** determines whose marks are shown. Flutter uses stub/mock until API is ready; stub should support loading, empty, and error states and "Marks not yet published" for at least one exam so acceptance criteria can be verified.

---

## Traceability

- **F1–F8, NF1–NF3**: [marks-markbooks-spec.md](marks-markbooks-spec.md)
- **Structure, components, states, copy, a11y**: [marks-ui-ux-design.md](marks-ui-ux-design.md)
- **Build order and refs**: [marks-ui-build-outline.md](marks-ui-build-outline.md)
