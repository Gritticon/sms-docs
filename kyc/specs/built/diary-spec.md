---
title: Diary Spec
type: spec
project: kyc
module: Diary
last-updated: 2026-03-18
---

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Diary — Specification

## 1. Summary
The Diary module allows students and parents to view daily teacher notes, homework, and other important class-specific updates. It replaces physical diaries and scattered messages with a centralized, date-organized view inside KYC.

All diary entries are managed and published in SMS; KYC presents them per student and date.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display a date-organized list of diary entries for the active student, including title, date, class/subject, and a brief description. | P0 |
| F2 | System shall allow users to select a date (e.g., today, previous days) to view diary entries for that date. | P0 |
| F3 | System shall allow users to open an entry to see full details (e.g., homework instructions, attachments where supported). | P0 |
| F4 | System shall differentiate between student- and parent-relevant notes where specified by the backend (e.g., “For parents”, “For students”). | P1 |
| F5 | System shall update displayed diary entries according to the active child for multi-child parents. | P0 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Diary views shall be responsive for easy reading on mobile and desktop. | P0 |
| NF2 | Daily diary content shall load quickly and be cached for short periods where appropriate. | P1 |

## 3. User Stories & Acceptance Criteria

### Story 1: Student checks today’s diary
**As a** student, **I want** to see today’s diary entries **so that** I know my homework and important notes.

**Acceptance criteria:**
- [ ] Given I am logged in as a student, when I open the Diary section, then I see entries for today if any exist.
- [ ] Given there are no entries for today, when I open the Diary section, then I see a clear “No diary entries for today” message.

**Requirement IDs:** F1, F2, NF1

---

### Story 2: Parent reviews past diary notes
**As a** parent, **I want** to review previous diary entries **so that** I can follow up on homework and communications.  

**Acceptance criteria:**
- [ ] Given I am logged in as a parent and have selected a child, when I open the Diary and change the selected date, then I see that child’s diary entries for that date.
- [ ] Given I open an entry, when I view its details, then I can read the full content and see whether it is tagged for parents, students, or both (if provided).

**Requirement IDs:** F1, F2, F3, F4, F5, NF1

## 4. Dependencies
- **SMS API diary endpoints**: Provide per-student diary entries by date and any tags/visibility rules.
- **Calendar/date utilities**: For date selection UI.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Large volumes of daily diary entries may be hard to scan. | Medium | Low | Use concise titles and allow drill-down into details; rely on SMS to keep entries manageable. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** Show clear messaging when no entries exist for a selected date or when the module is not enabled by the school.
- **Invalid input:** If a user chooses a date outside the academic year or a malformed date, reset to a valid default and inform the user softly.
- **Failure/offline:** On load failure, show an error with retry while preserving already loaded entries where possible.

## 7. MVP Scope

### In scope
- Per-day diary entry listing and detail view.
- Multi-child parent support.

### Out of scope (backlog)
- Direct replies or acknowledgements on diary entries from KYC.
- Teacher-side diary creation/editing (SMS-only).

### Success criteria for MVP
- Students and parents consistently use the Diary view for homework and daily notes instead of physical diaries and ad-hoc chat groups.

