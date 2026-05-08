---
title: Authentication and Account Management Spec
type: spec
project: kyc
module: Authentication
last-updated: 2026-03-18
---

> ⚠️ **Pipeline sync warning** (2026-04-08): This document is marked `built` but no
> matching implementation was detected during automated codebase scan. Verify manually.

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## Authentication & Account Management — Specification

## 1. Summary
The Authentication & Account Management module provides secure login and session handling for students and parents accessing KYC, scoped by school_id. It supports multi-child parent accounts with an easy account switcher and ensures that all subsequent module data (attendance, marks, fees, etc.) is displayed for the currently active child.

All credential verification, token management, and authorization logic is handled by the SMS API backend; KYC is responsible for presenting intuitive login flows, handling error states, and managing UI state around the authenticated user and selected child.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall provide a login screen for students and parents that accepts credentials defined by SMS API (e.g., username/ID and password, or other supported identifiers). | P0 |
| F2 | System shall associate every login request with a specific school_id so that authentication is performed in the correct school context. | P0 |
| F3 | System shall support both student and parent roles, showing role-appropriate navigation and data after successful login. | P0 |
| F4 | System shall receive and store session/auth tokens from SMS API on successful login, using secure frontend storage mechanisms as defined by the backend contract. | P0 |
| F5 | System shall provide an error message when login fails (e.g., invalid credentials, locked account) without revealing sensitive details. | P0 |
| F6 | System shall allow users to log out explicitly, clearing all stored session/auth tokens and user context from the frontend. | P0 |
| F7 | For parent accounts with multiple linked students, system shall display an account switcher that lists each child with basic identifying details (name, class/section). | P0 |
| F8 | System shall allow a parent to switch the active child; after switching, all KYC modules shall show data for the newly selected child. | P0 |
| F9 | System shall remember the last active child for a parent within a session so that refreshing the page does not reset the context unnecessarily. | P1 |
| F10 | System shall expose basic account information in an “Account” area (e.g., user name, role, active child, school name) and provide access to logout from there. | P0 |
| F11 | System shall handle session expiration by detecting invalid/expired tokens and redirecting the user to login with a clear message. | P0 |
| F12 | System shall prevent access to authenticated routes (e.g., Attendance, Marks, Fees) if the user is not logged in or the session is invalid. | P0 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | All authentication-related API calls shall be made over HTTPS to SMS API endpoints. | P0 |
| NF2 | Login and logout flows shall complete within 3 seconds under normal network conditions, excluding backend processing delays. | P1 |
| NF3 | Authentication UI shall be responsive and usable on mobile, tablet, and desktop screens. | P0 |
| NF4 | Error messages shall be clear and user-friendly, avoiding exposure of technical details (e.g., stack traces, internal error codes). | P0 |
| NF5 | Session handling code shall minimize storage of sensitive data on the client and follow best practices recommended by SMS API (e.g., HTTP-only cookies vs tokens in memory). | P0 |
| NF6 | Account switching interactions shall be lightweight and not require full re-authentication when the session is still valid. | P1 |

## 3. User Stories & Acceptance Criteria

### Story 1: Log in as a student
**As a** student, **I want** to log in securely **so that** I can access my personal academic information.

**Acceptance criteria:**
- [ ] Given I am on the KYC login screen for my school, when I enter valid student credentials and submit, then I am logged in and taken to the authenticated home view.
- [ ] Given I enter invalid credentials, when I submit, then I see a clear error message (e.g., “Incorrect username or password”) without revealing which field is incorrect.
- [ ] Given I am successfully logged in, when I navigate to any module, then my student identity (and school_id) is used for all API requests.

**Requirement IDs:** F1, F2, F3, F4, F5, F12, NF1, NF3, NF4

---

### Story 2: Log in as a parent
**As a** parent, **I want** to log in and access information for my child(ren) **so that** I can stay informed and manage school-related tasks.

**Acceptance criteria:**
- [ ] Given I am on the KYC login screen for my school, when I enter valid parent credentials and submit, then I am logged in and taken to the authenticated home view.
- [ ] Given I am a parent with a single linked child, when I log in, then the system automatically sets that child as the active context.
- [ ] Given I am a parent with multiple linked children, when I log in, then the system either prompts me to choose a child or defaults to one while clearly displaying the option to switch.

**Requirement IDs:** F1, F2, F3, F4, F7, F9, NF1, NF3, NF4

---

### Story 3: Switch between my children
**As a** parent with two or more children in the same school, **I want** to switch between my children’s profiles from one account **so that** I can view and manage each child’s information easily.

**Acceptance criteria:**
- [ ] Given I am logged in as a parent with multiple children, when I open the account switcher (from the Account area or top-level UI), then I see a list of my children with identifiable details (name, class/section).
- [ ] Given I select a different child in the account switcher, when the app reloads module views (e.g., Attendance, Marks, Fees, Diary), then data is updated for the newly selected child.
- [ ] Given I refresh the page while logged in, when the app reloads, then the previously active child remains active within the same session (subject to backend/token constraints).

**Requirement IDs:** F7, F8, F9, F10, NF3, NF6

---

### Story 4: Handle session expiration
**As a** user, **I want** to be logged out or prompted to log in again when my session expires **so that** my data remains secure and I understand what to do next.

**Acceptance criteria:**
- [ ] Given my token/session has expired, when I attempt to access an authenticated route or the app makes an API call, then I am redirected to the login screen or shown a prompt to log in again.
- [ ] Given I am redirected due to expiration, when I log in again successfully, then I am returned to a reasonable default view (e.g., Home or previously requested module, if safe to do so).
- [ ] Given my session has expired, when another user opens the app on the same browser, then they cannot access my data without logging in with their own credentials.

**Requirement IDs:** F4, F11, F12, NF1, NF5

---

### Story 5: Log out securely
**As a** user, **I want** to log out explicitly **so that** no one else can access my information from my device without logging in again.

**Acceptance criteria:**
- [ ] Given I am logged in, when I open the Account area and choose “Log out”, then all local session/auth tokens and user context are cleared.
- [ ] Given I log out successfully, when I navigate back or refresh the app, then I see a non-authenticated view (e.g., login or school selection) and cannot access any prior authenticated data.

**Requirement IDs:** F6, F10, F12, NF1, NF5

## 4. Dependencies
- **SMS API Authentication endpoints**: Provide login, logout (if applicable), token issuance/refresh, role detection (student vs parent), and error codes/messages.
- **SMS data model**: Maintains relationships between parents and students, including which children are linked to which parent account and which school_id(s) they belong to.
- **KYC routing and navigation**: Must support authenticated vs unauthenticated routes and route guards based on session state.
- **Secure storage mechanism**: Cookies, secure storage APIs, or in-memory token strategies as defined by backend security design.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Weak session handling or token storage could expose user data. | High | Medium | Follow backend security recommendations (e.g., HTTP-only cookies, minimal token exposure in JS), avoid localStorage for sensitive tokens where possible. |
| Complex multi-child handling may cause confusion in UX. | Medium | Medium | Use clear visual cues for the active child and keep switching interactions simple and consistent across modules. |
| Poor error messaging may lead to support overhead (users unsure why they cannot log in). | Medium | Medium | Standardize friendly, non-technical error messages and map common backend error codes to user-friendly text. |
| Session expiration during active use could interrupt flows unexpectedly. | Medium | Low | Implement token refresh where supported, and show clear “session expired” prompts with smooth re-login flows. |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If a parent or student account has no linked children (misconfiguration), show a clear message instructing them to contact the school rather than failing silently. If the account has only one linked child, hide or simplify the account switcher UI.
- **Invalid input:** On login, handle empty fields, unsupported characters, or malformed identifiers with inline validation messages. Avoid differentiating between “user not found” and “wrong password” to prevent enumeration.
- **Failure/offline:** When authentication endpoints are unavailable, display a generic “Unable to log in right now, please try again later” message and avoid infinite spinners. When network connectivity is lost mid-session, show a non-blocking notification and retry logic where appropriate.
- **Other:** Handle revoked or disabled accounts gracefully (e.g., “Your account has been disabled by the school”) based on backend error codes. For account switching, handle the removal of a child from a parent’s account during an active session by updating the list and redirecting to a valid context.

## 7. MVP Scope

### In scope
- Student and parent login tied to a specific school_id context.
- Role-aware navigation after login (student vs parent).
- Multi-child parent account switcher for children in the same school.
- Secure session handling in the frontend according to SMS API contract.
- Clear logout and session expiration experiences.

### Out of scope (backlog)
- Self-service registration, password reset, or account recovery flows (assumed to be handled by SMS or external identity provider for now).
- Social login (e.g., Google, Apple sign-in).
- Cross-school parent accounts with a unified cross-school switcher.

### Success criteria for MVP
- At least 95% of valid login attempts for pilot schools succeed without manual support intervention.
- Multi-child parents in pilot schools can successfully switch between children and access correct data in less than 3 taps/clicks.
- No major authentication-related security incidents or data exposure linked to KYC’s frontend handling during pilot usage.

