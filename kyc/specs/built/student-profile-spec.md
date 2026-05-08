---
title: Student Profile Spec
type: spec
project: kyc
module: Student Profile
last-updated: 2026-03-18
---

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Student Profile Management — Specification

## 1. Summary
The Student Profile Management module lets students and parents view key profile information for each student, such as name, class/section, roll number, contact details, and other school-defined fields. It centralizes identity and academic context that other modules (attendance, marks, fees, transport) depend on.

All profile data and update workflows are owned by the SMS backend; KYC presents a read-focused view and may provide request/change initiation mechanisms rather than direct edits.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display core identity information for the active student (name, photo/avatar if available, class, section, roll number, admission number) as provided by the backend. | P0 |
| F2 | System shall display additional school-configured profile fields (e.g., house, date of birth, contact details) as read-only values. | P1 |
| F3 | System shall allow a parent with multiple children to view each child’s profile when that child is active. | P0 |
| F4 | System shall clearly show the relationship between the logged-in parent and the student (e.g., “Father”, “Mother”, “Guardian”) if provided. | P1 |
| F5 | System may provide a way for users to request changes to certain profile fields (e.g., address, contact number) via a backend-defined mechanism (e.g., change request API or link), without directly editing fields in KYC. | P2 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Profile view shall be responsive and legible across mobile, tablet, and desktop. | P0 |
| NF2 | Profile data shall load quickly (within ~1–2 seconds) since it is small and cached where appropriate. | P1 |
| NF3 | Sensitive fields (e.g., personal contact details) shall be displayed only to authorized roles (typically parents, not other students) based on backend authorization. | P0 |

## 3. User Stories & Acceptance Criteria

### Story 1: Student views own profile
**As a** student, **I want** to view my core profile information **so that** I can verify what the school has on record.

**Acceptance criteria:**
- [ ] Given I am logged in as a student, when I open the Profile area, then I see my name, class/section, roll number, and other key identifiers from SMS.
- [ ] Given some optional profile fields are not filled in, when I view my profile, then those fields are clearly shown as empty or hidden rather than misleading default values.

**Requirement IDs:** F1, F2, NF1

---

### Story 2: Parent views each child’s profile
**As a** parent with two kids in the same school, **I want** to see each child’s profile **so that** I can confirm their details and context.  

**Acceptance criteria:**
- [ ] Given I am logged in as a parent and have multiple children linked, when I select a child via the account switcher and open Profile, then I see that child’s profile details.
- [ ] Given I switch to another child, when I re-open or refresh the Profile view, then I see the second child’s details instead of the first.
- [ ] Given my relationship to the student is provided by SMS, when I open the profile, then I see it clearly indicated (e.g., “Guardian: Mother”).

**Requirement IDs:** F1, F2, F3, F4, NF1, NF3

---

### Story 3: Request a change to profile information
**As a** parent, **I want** to request updates for certain profile details **so that** the school can correct or update its records.  

**Acceptance criteria:**
- [ ] Given I am viewing my child’s profile, when I see outdated information in changeable fields (e.g., address, phone number), then I have a clear way to request a change (e.g., “Request update” button or link) if supported by the backend.
- [ ] Given I submit a change request, when the request is accepted by the system, then I see confirmation feedback (e.g., “Your request has been submitted”) with no direct edits shown until approved and updated via SMS.

**Requirement IDs:** F2, F5, NF1

## 4. Dependencies
- **SMS API profile endpoints**: Provide student identity details, additional configured fields, relationships, and any allowed change-request mechanisms.
- **Authentication & account context**: Provide the active student context (for students themselves or selected child for parents).

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Exposing too many internal fields may confuse users. | Medium | Medium | Limit displayed fields to those that are meaningful to parents/students; rely on SMS configuration to control visibility. |
| Direct editing in KYC could conflict with school approval processes. | High | Medium | Restrict KYC to read-only views and change requests; keep approval and final updates in SMS. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If a profile is incomplete or missing data, show an explanatory message and encourage the user to contact the school if needed.
- **Invalid input:** For any change-request inputs (if implemented), validate basic formatting in the UI and rely on backend validation for final decisions.
- **Failure/offline:** If profile data fails to load, show an error message and offer retry; keep previously loaded profile visible when possible.

## 7. MVP Scope

### In scope
- Read-only per-student profile view with key identity and class information.
- Multi-child parent support for viewing each child’s profile.
- Basic indication of parent-student relationship where provided.

### Out of scope (backlog)
- Full self-service profile editing and approval workflows.
- Upload/management of profile pictures from KYC (treated as future enhancement or SMS feature).

### Success criteria for MVP
- Students and parents can verify core profile data without relying on separate school systems or paper records.
- Fewer basic profile information queries to school once KYC is adopted.

