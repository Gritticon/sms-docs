---
title: Specs Lifecycle and Migration Process
type: process
project: kyc
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Specs Lifecycle & Migration Process — KYC

This document defines how requirement specifications for KYC are managed between `kyc/docs/planned` and `kyc/docs/built`, with a focus on frontend (UI) behavior first and backend/API integration later.

### 1. Folder roles

- **`kyc/docs/planned`**  
  - Holds the latest **design-time specifications** for KYC features.  
  - Oriented primarily around **UI behavior, user stories, and UX flows**.  
  - Includes references to SMS API at a high level but does **not** lock down low-level API contracts (those will be defined in backend/API design docs).

- **`kyc/docs/built`**  
  - Holds **frozen snapshots** of specs that have been implemented and released to users.  
  - Reflects the exact scope of what is live in production at a given release/version.  
  - Used as a reference for QA, support, and regression analysis.

### 2. Per-feature lifecycle

For each module spec (e.g., `attendance-analytics-spec.md`, `fees-payments-spec.md`):

1. **Draft / Design (planned)**
   - Write or update the spec in `kyc/docs/planned`, focusing on:  
     - UI flows, states, and user stories (students, parents).  
     - Functional and non-functional requirements from the **frontend** perspective.  
     - High-level dependency notes on SMS API (e.g., “backend provides attendance summary per student”).  
   - Backend/API integration details (endpoints, payloads, error codes) are documented separately in SMS/SMS-API design docs.

2. **Implement UI**
   - Frontend team implements the UI in Flutter for the KYC module following the planned spec.  
   - API integration stubs may be created (mock data, fixtures) while backend contracts are finalized elsewhere.

3. **Integrate APIs**
   - Once SMS API contracts are ready, the frontend integrates with real endpoints, ensuring that:  
     - All **business logic and validation** remain on the backend.  
     - KYC uses minimal, well-defined response shapes and error mappings.  
   - Any changes discovered during integration that affect UX or requirements are updated in the **planned** spec.

4. **Validate & Release**
   - QA verifies implementation against the **planned** spec.  
   - Once a feature is confirmed as released (e.g., for a given version or school rollout), the spec is **snapshotted** to `kyc/docs/built`.

5. **Snapshot to built**
   - Copy the finalized spec file from `kyc/docs/planned` to `kyc/docs/built` under the same filename.  
   - Optionally add a short header or footer note indicating:  
     - Release/version identifier.  
     - Date of release.  
     - Any known deviations from the spec (if necessary).

6. **Evolve for future changes**
   - For future enhancements, **continue editing only the version in `kyc/docs/planned`**.  
   - Keep `kyc/docs/built` snapshots immutable, representing what is currently/previously live.  
   - When a new iteration of a feature is released, repeat steps 2–5 and either:  
     - Overwrite the previous snapshot in `built` with the updated one, or  
     - Keep multiple dated/numbered snapshots if release history tracking is required.

### 3. API integration planning (later phase)

- Detailed API-level design (endpoint URLs, request/response schemas, error handling) will be captured in **SMS/SMS-API-specific documents**, not in KYC UI specs.  
- Each KYC spec already references its backend dependencies at a high level, which can be used as input for:  
  - SMS-API endpoint design.  
  - Validation rules and business logic definitions.  
  - Performance and security considerations on the backend.

This ensures that all current documents in `kyc/docs/planned` are **UI-first**, ready for frontend implementation, while leaving room for a dedicated, structured planning phase for API integration on the backend side.

