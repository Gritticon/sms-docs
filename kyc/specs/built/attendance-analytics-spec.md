---
title: Attendance Analytics Spec
type: spec
project: kyc
module: Attendance
last-updated: 2026-03-18
---

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Attendance & Analytics — Specification

## 1. Summary
The Attendance & Analytics module allows students and parents to view daily and historical attendance data, understand patterns (e.g., monthly attendance rate), and quickly identify concerns. All attendance records and calculations are provided by the SMS API; KYC is responsible for presenting clear visualizations and summaries tailored to each student.

This module aims to replace manual tracking and reduce the information gap by making attendance information easily accessible anytime on web, tablet, and mobile.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display a daily/term-level attendance summary for the active student, including present, absent, and other statuses provided by the backend. | P0 |
| F2 | System shall allow users to view attendance over a selected date range (e.g., month, term, academic year) using backend-provided data. | P0 |
| F3 | System shall present attendance in a calendar or list view that clearly shows each day’s status. | P0 |
| F4 | System shall display aggregated attendance metrics such as total days present, total days absent, and overall attendance percentage for the selected period. | P0 |
| F5 | System shall support subject-wise or period-wise attendance views if provided by the backend (e.g., per class/period status), otherwise default to daily-level only. | P1 |
| F6 | System shall update attendance data based on the active child for multi-child parents, reflecting the selected student’s data. | P0 |
| F7 | System shall surface high-level alerts for critical thresholds (e.g., low attendance) if indicated by backend flags. | P1 |
| F8 | System shall allow navigation between months/terms via simple controls (e.g., previous/next, date picker). | P0 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Attendance views shall be responsive and readable on mobile, tablet, and desktop. | P0 |
| NF2 | Attendance data for a typical month shall load within 2–3 seconds on a standard school network. | P1 |
| NF3 | Visual representations (e.g., color coding for present/absent) shall be accessible and not rely on color alone (use legends, labels). | P0 |
| NF4 | The module shall handle large academic years by paginating or chunking data (e.g., monthly segments) to avoid performance issues. | P1 |

## 3. User Stories & Acceptance Criteria

### Story 1: View my recent attendance
**As a** student, **I want** to see my recent attendance **so that** I know how regularly I am attending school.

**Acceptance criteria:**
- [ ] Given I am logged in as a student, when I open the Attendance section, then I see my attendance for the current month (or default period) with each day’s status.
- [ ] Given I am on the Attendance view, when I move to the previous or next month/period, then the data updates accordingly.
- [ ] Given I view my attendance summary, when I check the summary metrics, then I can see total days present, total days absent, and overall percentage for the selected period.

**Requirement IDs:** F1, F2, F3, F4, F8, NF1

---

### Story 2: Parent monitors child attendance
**As a** parent, **I want** to monitor my child’s attendance and trends **so that** I can take action if attendance drops.  

**Acceptance criteria:**
- [ ] Given I am logged in as a parent and have selected a child, when I open the Attendance section, then I see that child’s attendance data and summary metrics.
- [ ] Given I switch to another child, when I return to Attendance, then the data updates to show the newly selected child’s attendance.
- [ ] Given the backend flags low attendance or concerning patterns for my child, when I open Attendance, then I see a clear visual indicator or message highlighting the concern.

**Requirement IDs:** F1, F2, F4, F6, F7, NF1

---

### Story 3: View detailed attendance for a specific day or subject
**As a** student or parent, **I want** to see more details for a specific day or subject **so that** I understand why a particular status was recorded.  

**Acceptance criteria:**
- [ ] Given I am viewing the attendance calendar/list, when I select a specific day, then I see detailed information (e.g., subject-wise/period-wise status, remarks) if provided by the backend.
- [ ] Given subject-wise data is not enabled for the school, when I select a day, then I see a simplified detail view that at least shows the overall status and any available notes.

**Requirement IDs:** F3, F5, NF1

## 4. Dependencies
- **SMS API attendance endpoints**: Provide daily and aggregated attendance data, per-student, per-period/subject (if applicable), and any risk flags for low attendance.
- **Student profile and relationships**: Determine which students are linked to each parent and which student is active in KYC.
- **Calendar/date utilities**: Used by KYC to handle date ranges and display calendars consistently.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Very large attendance histories may cause slow loads or heavy payloads. | Medium | Medium | Paginate by month/term, lazy-load additional periods on demand, and request minimal data needed for summaries. |
| Inconsistent attendance policies across schools may cause confusion. | Medium | Medium | Use generic status labels and rely on back-end-provided descriptions; allow schools to configure explanations via SMS. |
| Color-only status indicators may reduce accessibility. | Medium | Low | Always pair colors with status labels or icons and provide legends. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If there is no attendance data yet (e.g., new academic year or new student), show a meaningful message such as “No attendance records available yet” instead of a blank screen.
- **Invalid input:** If a user selects an invalid date range (e.g., start after end), reset to a valid default and show a small helper message.
- **Failure/offline:** If attendance data cannot be loaded due to network or server errors, show a message with a retry option, and keep previously loaded data visible if available.
- **Other:** Handle time zone differences (if any) by using backend-provided date/times and school-level local rules instead of client-side assumptions.

## 7. MVP Scope

### In scope
- Monthly/period-based attendance views per student.
- Summary metrics for attendance percentage and counts.
- Basic alerts for low attendance where backend provides flags.
- Mobile-friendly responsive UI.

### Out of scope (backlog)
- Complex analytics and charts beyond simple summaries (e.g., year-over-year comparison charts).
- Customizable attendance policies or labels at the frontend (treated as SMS configuration/back-end concern).
- Attendance editing or justification flows from KYC (e.g., requesting leave or corrections).

### Success criteria for MVP
- Parents and students in pilot schools can quickly find and interpret attendance data with minimal guidance.
- A measurable decrease in parent queries to school regarding basic attendance questions after KYC rollout.

