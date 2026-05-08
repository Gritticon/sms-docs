---
title: Marks and Markbooks Spec
type: spec
project: kyc
module: Marks
last-updated: 2026-03-18
---

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Marks & Markbooks — Specification

## 1. Summary
The Marks & Markbooks module enables students and parents to view academic performance across exams, subjects, and terms. It provides clear visibility into individual marks, overall grades, and high-level trends so families can understand progress and take timely action.

All marks, grading logic, and calculations are handled by the SMS API. KYC focuses on intuitive presentation, navigation across exams/terms, and per-student views (including multi-child support for parents).

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display a list of exams/assessments (e.g., unit tests, midterms, finals) for the active student, as provided by the backend. | P0 |
| F2 | System shall display subject-wise marks for a selected exam, including marks obtained, maximum marks, and any grade or remark fields from the backend. | P0 |
| F3 | System shall show overall performance indicators for each exam (e.g., total marks, percentage, grade) based on backend-calculated values. | P0 |
| F4 | System shall allow users to switch between different exams/terms for the same academic year. | P0 |
| F5 | System shall handle scenarios where marks are not yet published for an exam by clearly indicating “Not published” or equivalent status. | P0 |
| F6 | System shall update marks data based on the active child for multi-child parents. | P0 |
| F7 | System shall optionally show basic performance trends over time (e.g., list of exams with percentages) if provided by backend analytics. | P1 |
| F8 | System shall highlight subjects or exams flagged by the backend as areas of concern (e.g., failing grades). | P1 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Marks views shall be readable and scrollable on mobile, tablet, and desktop with responsive layouts. | P0 |
| NF2 | Loading marks for a typical exam shall complete within 2–3 seconds under normal conditions. | P1 |
| NF3 | Markbook tables and summaries shall be accessible, with clear labels and not relying solely on color. | P0 |

## 3. User Stories & Acceptance Criteria

### Story 1: View my marks for an exam
**As a** student, **I want** to see my marks for a specific exam **so that** I understand my performance subject by subject.

**Acceptance criteria:**
- [ ] Given I am logged in as a student, when I open the Marks section, then I see a list of my exams/assessments for the current academic period (as provided by the backend).
- [ ] Given I select an exam from the list, when the marks data loads, then I see a subject-wise breakdown including marks obtained, maximum marks, and any grade/remark fields.
- [ ] Given the backend has not yet published marks for an exam, when I select that exam, then I see a clear message such as “Marks not yet published” instead of empty or misleading data.

**Requirement IDs:** F1, F2, F3, F4, F5, NF1

---

### Story 2: Parent reviews child performance
**As a** parent, **I want** to review my child’s marks across exams **so that** I can support their learning.  

**Acceptance criteria:**
- [ ] Given I am logged in as a parent and have selected a child, when I open the Marks section, then I see that child’s exam list and mark details.
- [ ] Given I switch to a different child via the account switcher, when I open Marks, then the data updates to show the newly selected child’s marks.
- [ ] Given the backend flags concerning performance (e.g., failing subjects), when I view marks, then those subjects or exams are visually highlighted with a clear label or icon.

**Requirement IDs:** F1, F2, F3, F6, F8, NF1

---

### Story 3: Understand overall performance over time
**As a** parent or student, **I want** to see overall performance across multiple exams **so that** I can understand long-term trends.  

**Acceptance criteria:**
- [ ] Given I have marks available for multiple exams, when I view the exam list, then I see aggregated indicators such as percentage or grade per exam (where provided by the backend).
- [ ] Given the backend provides trend or summary analytics, when I open a performance overview view, then I see a simple representation (e.g., list or basic chart) summarizing changes over time.

**Requirement IDs:** F3, F4, F7, NF1, NF3

## 4. Dependencies
- **SMS API marks endpoints**: Provide exam lists, subject-wise marks, maximum marks, grades, remarks, and any analytics flags.
- **Academic term configuration**: Determine how exams are grouped (terms/semesters) and in what order they appear.
- **Student-parent relationships and active child context**: Ensures the correct student’s marks are loaded.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Complex grading schemes (e.g., different scales, weighting) may be hard to represent clearly in UI. | Medium | Medium | Rely on backend-calculated summaries and grades; keep UI focused on presenting values and descriptive labels from SMS. |
| Large markbooks across many exams may cause performance issues. | Medium | Medium | Load data per exam or in small batches; avoid loading all exam details at once. |
| Misinterpretation of partial or unpublished data by users. | Medium | Medium | Clearly label unpublished, partial, or provisional results in the UI and avoid showing incomplete aggregates. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If the student has no marks yet (e.g., new admission or pre-exam period), show a friendly “No marks available yet” message with context.
- **Invalid input:** If an invalid exam ID or term is requested (due to navigation or deep-link errors), show a generic error and return the user to a safe default exam list.
- **Failure/offline:** If marks cannot be loaded due to network or server issues, show an error with retry, while preserving any already loaded data on screen.
- **Other:** Handle exams that have only some subjects graded while others are pending by visually indicating pending subjects.

## 7. MVP Scope

### In scope
- Per-exam subject-wise marks view for each student.
- Basic overall performance indicators (total marks, percentages, grades) from backend.
- Simple per-exam list with aggregated indicators.
- Multi-child support for parents.

### Out of scope (backlog)
- Advanced analytical dashboards (e.g., comparative class averages, ranking views).
- Detailed grading scheme configuration UI (weights, custom scales) in KYC.
- Downloadable transcripts/report cards (covered in Documents module instead).

### Success criteria for MVP
- Students and parents can reliably find and understand marks for each published exam without requesting manual grade sheets from the school.
- Reduction in basic “marks inquiry” calls/messages to the school once KYC is adopted.

