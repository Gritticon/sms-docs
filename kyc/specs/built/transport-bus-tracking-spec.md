---
title: Transport and Bus Tracking Spec
type: spec
project: kyc
module: Transport
last-updated: 2026-03-18
---

> ⚠️ **Pipeline sync warning** (2026-04-08): This document is marked `built` but no
> matching implementation was detected during automated codebase scan. Verify manually.

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Transport / Track School Bus — Specification

## 1. Summary
The Transport module lets students and parents view transport assignments (bus, route, pickup/drop points) and, where available, an approximate live location or status of the school bus. It aims to improve safety and reduce uncertainty around pickup and drop timings.

All route definitions, GPS integration, and location updates are managed by SMS backend or integrated services; KYC presents read-only, student-specific transport information and basic status views.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall display transport assignment details for the active student, including bus/vehicle identifier, route name, pickup and drop points, and scheduled times as provided by the backend. | P0 |
| F2 | System shall indicate whether the student currently uses transport or not (e.g., “No transport assigned” state). | P0 |
| F3 | Where GPS tracking is enabled, system shall show a basic bus status view (e.g., current/last known location on a map or simplified status such as “On route”, “At school”), based on backend-provided data. | P1 |
| F4 | System shall update transport information based on the active child for multi-child parents. | P0 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | Transport views shall be responsive and clear on mobile, tablet, and desktop, using map or non-map layouts depending on what is feasible. | P0 |
| NF2 | Location updates (where available) shall be refreshed at reasonable intervals as defined by backend and without excessive polling from the frontend. | P1 |

## 3. User Stories & Acceptance Criteria

### Story 1: See my bus assignment
**As a** parent, **I want** to see my child’s transport details **so that** I know where they are picked up and dropped off.

**Acceptance criteria:**
- [ ] Given my child uses school transport, when I open the Transport section, then I see the assigned bus/route, pickup/drop points, and scheduled times.
- [ ] Given my child does not use transport, when I open the Transport section, then I see a clear “No transport assigned” message rather than an empty view.

**Requirement IDs:** F1, F2, F4, NF1

---

### Story 2: View bus status (if enabled)
**As a** parent, **I want** to see whether the bus is on its way or at school **so that** I can plan drop-off and pickup times.

**Acceptance criteria:**
- [ ] Given GPS-based tracking is enabled and available from the backend, when I open the Transport status view, then I see at least a simplified status (e.g., “On route to school”, “Near pickup point”) or last-known location visualization.
- [ ] Given GPS data is not available or temporarily unavailable, when I open the status view, then I see a clear message indicating that live tracking is not available.

**Requirement IDs:** F3, NF1, NF2

## 4. Dependencies
- **SMS API transport endpoints**: Provide per-student transport assignments and status.
- **GPS/location services (backend)**: Integrate with buses and expose simplified location or status to SMS API.
- **Student-parent relationships and active child context**: Determine which transport details to display.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Inaccurate or delayed GPS data could mislead parents. | High | Medium | Present location/status as approximate, rely on backend to provide quality data, and include disclaimers where appropriate. |
| Map-heavy UI may be slow on older devices or networks. | Medium | Medium | Provide a fallback non-map status view and optimize map rendering; avoid over-refreshing. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** When transport is not enabled or not assigned, show clear messaging instead of blank UI.
- **Invalid input:** If an invalid route or bus ID is referenced, show an error and fallback to a safe default message.
- **Failure/offline:** If transport data fails to load or update, display an error with retry; for GPS failures, fall back to the last known safe non-live data.

## 7. MVP Scope

### In scope
- Read-only transport assignment view per student.
- Basic bus status indication when backend provides it.

### Out of scope (backlog)
- Real-time, high-frequency map tracking with advanced features (e.g., ETA calculation).
- Route planning or changes initiated from KYC.

### Success criteria for MVP
- Parents can reliably find their child’s transport assignment and basic bus status (where available) in KYC.

