---
title: Communication Spec
type: spec
project: kyc
module: Communication
last-updated: 2026-03-18
---

> ⚠️ **Pipeline sync warning** (2026-04-08): This document is marked `built` but no
> matching implementation was detected during automated codebase scan. Verify manually.

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Communication (Messaging) — Specification

## 1. Summary
The Communication module provides structured, one-way or limited two-way messaging between the school and parents/students within KYC. It is intended for targeted messages that do not fit into broad notifications, complaints, or diary entries, while avoiding full chat complexity.

All message creation, routing, and permission logic are owned by SMS; KYC focuses on presenting messages, allowing simple replies where enabled, and linking to related modules.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display a list of messages addressed to the logged-in user (and/or their children), including subject, sender (e.g., teacher, admin), and date/time. | P0 |
| F2 | System shall allow users to open a message to view its full content and any structured metadata (e.g., related student, class, module). | P0 |
| F3 | System shall mark messages as read/unread where supported by the backend and visually indicate read state. | P1 |
| F4 | System may allow simple replies or acknowledgments to messages if enabled by the backend, using a constrained input (e.g., short text or yes/no acknowledgement). | P2 |
| F5 | System shall ensure that messages are scoped correctly to the active child or to the parent account, depending on the message type. | P0 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Message listing and detail views shall be responsive and easy to scan on mobile and desktop. | P0 |
| NF2 | The UI shall be intentionally simple and not attempt to mimic a full chat app (no typing indicators, presence, etc.). | P0 |

## 3. User Stories & Acceptance Criteria

### Story 1: View messages from school
**As a** parent, **I want** to see messages sent to me by the school **so that** I stay informed beyond general announcements.

**Acceptance criteria:**
- [ ] Given I am logged in, when I open the Communication section, then I see a list of messages addressed to my account and/or my children, sorted by date/time.
- [ ] Given I tap/click a message, when I open it, then I see the full content and any related information (e.g., which child it refers to).

**Requirement IDs:** F1, F2, F5, NF1

---

### Story 2: Acknowledge or reply (where allowed)
**As a** parent, **I want** to acknowledge certain messages **so that** the school knows I have seen them.

**Acceptance criteria:**
- [ ] Given a message requires acknowledgment and the backend supports it, when I view the message, then I see a clear action (e.g., “Acknowledge”) and can trigger it once.
- [ ] Given I acknowledge a message successfully, when I revisit the message, then its status reflects that it has been acknowledged.

**Requirement IDs:** F3, F4, NF1

## 4. Dependencies
- **SMS API messaging endpoints**: Provide message lists, content, read/unread state, and optional reply/acknowledgement actions.
- **Notifications module**: May notify users of new messages and link to the Communication module.
- **Student-parent relationships**: Determine which messages are visible to whom.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Pressure to expand this module into a full chat system. | Medium | Medium | Clearly scope this as structured messaging; rely on SMS for any richer interaction, if ever required. |
| Confusion between Complaints vs Communication. | Medium | Medium | Use clear labels and guidance: Complaints for issues to resolve; Communication for general targeted messages. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If there are no messages, show a simple “No messages yet” state.
- **Invalid input:** For replies or acknowledgements, validate local inputs and handle backend failures gracefully (e.g., show error and allow retry).
- **Failure/offline:** If message listing or loading fails, show error and retain any locally cached messages while indicating they may be outdated.

## 7. MVP Scope

### In scope
- Read-only message list and detail view.
- Basic read/unread indication where supported.
- Simple acknowledgements for certain messages.

### Out of scope (backlog)
- Real-time chat features (typing, presence, continuous threads like chat apps).
- Multimedia messaging beyond simple text and existing attachments supported via SMS.

### Success criteria for MVP
- Parents use the Communication module for targeted messages instead of unmanaged chat groups.
- Schools see improved clarity in one-way and structured communications to parents.

