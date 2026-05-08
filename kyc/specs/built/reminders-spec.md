---
title: Reminders Spec
type: spec
project: kyc
module: Reminders
last-updated: 2026-03-18
---

> ⚠️ **Pipeline sync warning** (2026-04-08): This document is marked `built` but no
> matching implementation was detected during automated codebase scan. Verify manually.

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Reminders — Specification

## 1. Summary
The Reminders module ensures that parents and students receive timely prompts about important school-related actions, primarily fee payments in the initial phase, and potentially other critical deadlines later (e.g., form submissions, events). KYC focuses on displaying and managing reminder preferences at the UI level, while SMS backend schedules and sends actual reminder events.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display fee-related reminders for the active child (e.g., upcoming due date, overdue payments) using data or events from SMS API. | P0 |
| F2 | System shall surface reminders in relevant areas of the app (e.g., Home, Fees section, Notifications center) to draw attention to pending actions. | P0 |
| F3 | System shall allow users to see a simple list or history of reminders that have been sent or are active, where such data is provided by the backend. | P1 |
| F4 | System shall honor backend-configured reminder rules (schedules, frequency) and shall not create its own reminder logic in the frontend. | P0 |
| F5 | System may expose simple preference toggles (e.g., enable/disable certain reminder types) where supported by the backend. | P2 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Reminder indicators (badges, banners, list items) shall be visible and readable on mobile, tablet, and desktop without being overly intrusive. | P0 |
| NF2 | Reminder displays shall remain in sync with the underlying state (e.g., once a fee is paid, associated reminders should no longer show as active after backend update). | P0 |

## 3. User Stories & Acceptance Criteria

### Story 1: See fee payment reminders
**As a** parent, **I want** to see reminders about upcoming or overdue fees **so that** I don’t miss important payments.

**Acceptance criteria:**
- [ ] Given there are upcoming or overdue fee items for my child, when I open KYC (Home or Fees section), then I see clear reminder indicators about those fees.
- [ ] Given I pay the relevant fees and the backend updates their status, when I return to KYC and data is refreshed, then those reminders no longer show as active.

**Requirement IDs:** F1, F2, F4, NF1, NF2

---

### Story 2: View reminder list
**As a** parent, **I want** to see a list of reminders that have been sent or scheduled **so that** I understand what the system has already told me.  

**Acceptance criteria:**
- [ ] Given the backend exposes reminder history or active reminders, when I open the Reminders section, then I see a list with type (e.g., “Fee due”), date/time, and associated child (if applicable).

**Requirement IDs:** F1, F3, NF1

## 4. Dependencies
- **SMS API reminders/notifications endpoints**: Provide reminder events, types, associated entities (fees, events, forms), and any preference controls.
- **Fees & Invoices module**: Supplies fee status that drives fee reminder logic on the backend.
- **Notifications & Events module**: May share infrastructure or UI surfaces for displaying reminders.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Overuse of reminders could annoy users. | Medium | Medium | Keep frontend reminders concise; allow backend to control frequency; consider simple opt-out where feasible. |
| Out-of-sync reminders (e.g., fee already paid but reminder still visible). | Medium | Medium | Ensure KYC refreshes data from SMS after key actions and uses backend as the source of truth for reminder state. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If there are currently no reminders, show a friendly “No active reminders” message rather than an empty list.
- **Invalid input:** For any preferences UI (if implemented), validate selections and rely on backend for final enforcement.
- **Failure/offline:** If reminder data cannot be fetched, show a generic error message but do not block access to other modules.

## 7. MVP Scope

### In scope
- Display of fee-related reminders provided by SMS backend.
- Surfacing reminders in key UI areas (Home and Fees sections, and/or Notifications center).

### Out of scope (backlog)
- Complex per-user reminder scheduling or custom rules managed from KYC.
- Full preference management UI beyond basic enable/disable toggles.

### Success criteria for MVP
- Measurable reduction in late fee payments in pilot schools due to timely digital reminders.
- Positive feedback from parents that reminders are helpful but not overwhelming.

