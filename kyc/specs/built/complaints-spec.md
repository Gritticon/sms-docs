---
title: Complaints Spec
type: spec
project: kyc
module: Complaints
last-updated: 2026-03-18
---

> ⚠️ **Pipeline sync warning** (2026-04-08): This document is marked `built` but no
> matching implementation was detected during automated codebase scan. Verify manually.

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Complaints — Specification

## 1. Summary
The Complaints module allows parents (and optionally students, if permitted) to raise concerns or complaints to the school, track their status, and view responses. It replaces informal, hard-to-track channels with a simple, structured flow integrated into KYC.

Complaint handling, routing, and resolution workflows are owned by SMS; KYC provides the UI to submit and follow complaints.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall allow authorized users (primarily parents) to create a new complaint by selecting a category and entering a description. | P0 |
| F2 | System shall allow users to optionally associate a complaint with a specific child if they have multiple children. | P0 |
| F3 | System shall allow users to view a list of their submitted complaints with key details (category, date, status). | P0 |
| F4 | System shall allow users to open a complaint to see full details and any school responses or status updates. | P0 |
| F5 | System shall surface complaint status updates (e.g., Open, In Progress, Resolved) as provided by the backend. | P0 |
| F6 | System may allow users to add follow-up comments or clarifications to an existing complaint where supported by the backend. | P2 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Complaint creation and listing UIs shall be simple and mobile-friendly. | P0 |
| NF2 | Sensitive complaint content shall only be visible to the submitting user and authorized school staff, enforced primarily by the backend. | P0 |

## 3. User Stories & Acceptance Criteria

### Story 1: Raise a complaint
**As a** parent, **I want** to raise a complaint to the school **so that** issues can be tracked and addressed.

**Acceptance criteria:**
- [ ] Given I am logged in as a parent, when I open the Complaints section and choose “New complaint”, then I can select a category, optionally select a child, and enter a description.
- [ ] Given I submit a valid complaint form, when the backend confirms creation, then I see a confirmation and the complaint appears in my complaint list.

**Requirement IDs:** F1, F2, F3, NF1

---

### Story 2: Track complaint status
**As a** parent, **I want** to see the status of my complaints **so that** I know whether the school is acting on them.  

**Acceptance criteria:**
- [ ] Given I have submitted complaints, when I open the Complaints section, then I see a list of my complaints with at least category, date, and current status.
- [ ] Given a complaint’s status changes in SMS, when I refresh or revisit the list, then I see the updated status in KYC.
- [ ] Given I open a specific complaint, when I view its details, then I can see school responses or notes if provided by the backend.

**Requirement IDs:** F3, F4, F5, NF1

## 4. Dependencies
- **SMS API complaints endpoints**: Handle complaint creation, listing, and status updates, and optionally comments/responses.
- **Student-parent relationships and permissions**: Govern who can file complaints and link them to which students.
- **Notifications module**: May notify parents of responses or status changes.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Sensitive content may require careful access control. | High | Medium | Rely on backend for strict authorization and avoid exposing complaint IDs or details beyond the submitting account. |
| High volume of complaints may strain school workflows. | Medium | Medium | Start with simple flows and allow schools to manage internal processes in SMS; KYC only surfaces status. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If no complaints have been submitted, show a message such as “You have not submitted any complaints yet” with a clear CTA to create one.
- **Invalid input:** Validate basic fields (e.g., non-empty description) in the UI; handle backend validation failures with clear error messages.
- **Failure/offline:** If complaint creation or listing fails, show error messages and preserve unsent complaint content where possible so users can retry.

## 7. MVP Scope

### In scope
- Basic complaint creation with category and description.
- Association of complaints with a specific child.
- Complaint listing and detail view with status.

### Out of scope (backlog)
- Rich conversation threads or chat-like interfaces for complaints.
- Attachment uploads beyond simple text (can be future extension if backend supports it).

### Success criteria for MVP
- Parents use KYC Complaints instead of informal channels for school-related issues.
- Schools can track complaints more effectively via SMS due to structured inputs from KYC.

