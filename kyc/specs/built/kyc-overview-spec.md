---
title: KYC Overview Spec
type: spec
project: kyc
last-updated: 2026-03-18
---

> ⚠️ **Pipeline sync warning** (2026-04-08): This document is marked `built` but no
> matching implementation was detected during automated codebase scan. Verify manually.

> **Built snapshot** — 2025-03-03 — UI implemented; API integration may be pending.

## KYC Overview — Specification

## 1. Summary
KYC (Know Your Child) is a responsive, web-based and mobile-accessible portal that gives students and parents real-time visibility into school information including attendance, marks, timetable, fees, transport, announcements, and more. The portal consumes APIs from the SMS (School Management System) backend so that all core business logic, validation, and data processing remain server-side, while KYC focuses on a modern, intuitive UI that bridges the communication gap between schools and families.

KYC supports both student and parent roles and is designed to work across web, tablet, and mobile (via Flutter web and Flutter webview), enabling multi-child parents to manage all of their children from a single application using a school-specific whitelabeled experience.

## 2. Atomic Requirements

### 2.1 Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| F1 | System shall allow users to select a school context using a school identifier (school_id) before or during authentication so that the correct whitelabeled KYC instance is loaded. | P0 |
| F2 | System shall support two primary roles: Student and Parent, each with role-appropriate navigation and permissions. | P0 |
| F3 | System shall provide a responsive home/landing experience that shows a feed of school posts/events (Instagram-style), accessible after login. | P0 |
| F4 | System shall provide high-level navigation to the core KYC modules (Home, Transport, Diary, Account, plus others via menus) from every page. | P0 |
| F5 | System shall support parent accounts with multiple children enrolled at the same school, allowing quick account switching (Gmail-style account switcher). | P0 |
| F6 | System shall display per-child contextualized data (attendance, marks, timetable, fees, etc.) based on the currently selected child. | P0 |
| F7 | System shall consume all data via SMS API endpoints (FastAPI backend) and shall not implement business logic or data storage in the frontend. | P0 |
| F8 | System shall restrict access to student data to authenticated and authorized users only, based on role and school_id. | P0 |
| F9 | System shall surface key communication content (notifications, events, diary entries, fee reminders) prominently to reduce information gaps between school and parents/students. | P0 |
| F10 | System shall support localized time and date formats based on school configuration where provided by the backend. | P1 |
| F11 | System shall allow users to log out and clear local session data reliably across devices. | P0 |

### 2.2 Non-Functional
| ID | Requirement | Priority |
|----|-------------|----------|
| NF1 | KYC shall be implemented as a Flutter web frontend that runs in modern desktop and mobile browsers and inside Flutter mobile apps using a webview. | P0 |
| NF2 | KYC UI shall be responsive across mobile, tablet, and desktop screen sizes, using adaptive layouts and breakpoints. | P0 |
| NF3 | Average authenticated page load (after initial login) shall complete within 3 seconds on a typical school network connection. | P1 |
| NF4 | All API communication shall be performed over HTTPS with secure handling of authentication tokens as defined by SMS API. | P0 |
| NF5 | KYC shall not store sensitive data (e.g., passwords, full payment information) in the frontend beyond what is strictly necessary for the current session. | P0 |
| NF6 | Navigation structure and labels shall be consistent across devices, with bottom or side navigation adjusted per form factor (e.g., bottom navigation on mobile: Home, Transport, Diary, Account). | P0 |
| NF7 | KYC shall support at least the latest two major versions of Chrome, Safari, Edge, and Firefox on desktop, and modern WebView/WebKit/Chrome views on mobile. | P1 |
| NF8 | The system shall degrade gracefully when some modules are disabled for a school (e.g., hide links or show meaningful “Not enabled by your school” states). | P1 |
| NF9 | KYC shall be designed such that UI text and layout can be localized in the future (i18n-ready), even if only one language is supported in MVP. | P2 |

## 3. User Stories & Acceptance Criteria

### Story 1: Access KYC for my school
**As a** parent or student, **I want** to access the correct school-branded KYC portal using a school identifier **so that** I see information and branding specific to my school.

**Acceptance criteria:**
- [ ] Given I know my school’s identifier or access URL, when I open KYC, then I see my school’s name/logo or branding as provided by SMS configuration.
- [ ] Given I open KYC without a selected school, when I provide or select a school_id, then the app loads configuration (logo, colors if applicable, enabled modules) for that school.
- [ ] Given my school has disabled certain KYC modules, when I navigate the app, then those modules are hidden or clearly marked as unavailable.

**Requirement IDs:** F1, F4, F7, NF1, NF2, NF8

---

### Story 2: Use KYC as a student
**As a** student, **I want** to log in and see my core academic information in one place **so that** I can keep track of my attendance, marks, timetable, and school communications.

**Acceptance criteria:**
- [ ] Given I am a student with valid credentials, when I log in, then I see a home view showing recent posts/notifications from my school.
- [ ] Given I am logged in as a student, when I navigate using the main navigation, then I can access modules such as Attendance, Marks, Timetable, Diary, Notifications, and Events according to school configuration.
- [ ] Given I am viewing any module, when I use the app on different devices (mobile, tablet, desktop), then the layout adapts while keeping all essential information accessible.

**Requirement IDs:** F2, F3, F4, F6, F9, NF1, NF2

---

### Story 3: Use KYC as a multi-child parent
**As a** parent with multiple children in the same school, **I want** to switch between my children’s profiles within a single app **so that** I can manage all their information without logging in and out repeatedly.

**Acceptance criteria:**
- [ ] Given I am a parent linked to multiple students, when I open KYC, then I see which child is currently active in the UI (e.g., avatar/name at the top or in the account switcher).
- [ ] Given I am a multi-child parent, when I open the account switcher, then I can see a list of my children and select one to switch the context.
- [ ] Given I switch from Child A to Child B, when the app reloads module views (e.g., Attendance, Marks, Fees), then data updates to reflect Child B’s information.

**Requirement IDs:** F2, F5, F6, F9, NF2

---

### Story 4: Stay informed via the home feed
**As a** parent or student, **I want** to see recent school posts/events in a scrollable feed **so that** I am always up to date with what is happening at school.

**Acceptance criteria:**
- [ ] Given I am logged in, when I open the Home tab, then I see a vertically scrollable feed of posts with images and basic details (date, title, short description) in reverse chronological order.
- [ ] Given I view a post in the feed, when I interact with its action button, then I can share it via OS-level share intents (where supported) without likes/comments functionality.
- [ ] Given there are no posts from my school, when I open the Home tab, then I see a friendly empty state explaining that there are no recent events yet.

**Requirement IDs:** F3, F4, F9, NF2

---

### Story 5: Log out safely
**As a** parent or student, **I want** to log out and clear my session **so that** my information is not accessible to others who use my device.

**Acceptance criteria:**
- [ ] Given I am logged in, when I tap/click on the Account area and choose “Log out”, then I am signed out and returned to a non-authenticated view (e.g., login or school selection screen).
- [ ] Given I have logged out, when someone else opens the app on the same browser, then they cannot access my data without logging in again.

**Requirement IDs:** F4, F8, F11, NF4, NF5

## 4. Dependencies
- **SMS (School Management System)**: Provides configuration for each school (branding, enabled modules, API endpoints, roles, and relationships between parents and students).
- **SMS API (FastAPI backend)**: Exposes authenticated endpoints for all data KYC needs (attendance, marks, fees, transport, notifications, etc.) and handles business logic, validation, security, and persistence.
- **Authentication provider**: SMS API (or an integrated auth service) must implement login, token issuance/refresh, and role/permission verification.
- **Hosting and CDN**: KYC Flutter web build must be hosted on suitable infrastructure with HTTPS enabled and reasonable global performance for target schools.

## 5. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Inconsistent configuration across schools leads to unexpected UI states in KYC. | High | Medium | Define clear contracts for SMS configuration APIs and ensure KYC handles disabled/misconfigured modules with safe defaults and clear messaging. |
| Performance issues on low-end devices or slow networks reduce adoption. | Medium | Medium | Optimize Flutter web bundles, lazy-load heavy modules where possible, and cache non-sensitive data according to backend rules. |
| Complexity of multi-child parent flows causes user confusion. | Medium | Medium | Use a simple, Gmail-like account switcher with clear labels (child name, class) and consistent visual context indicators. |
| Browser compatibility issues with school lab environments. | Medium | Low | Target modern browsers explicitly; provide guidance to schools on supported browsers and versions. |
| Security misconfigurations in SMS API affect KYC data protection. | High | Low | Enforce HTTPS, adhere to JWT/session best practices, and rely on SMS API for access control; add frontend checks only as defense-in-depth (not primary security). |

## 6. Edge Cases & Error Handling
- **Empty/zero state:** If a module (e.g., Posts, Notifications) has no data, show a clear empty state message and guidance rather than an empty screen. If a module is disabled for a school, hide the entry or show an explanatory message when accessed.
- **Invalid input:** If an invalid school_id or access URL is used, show an error and allow the user to re-enter/select the correct school; avoid exposing information about other schools. If a user attempts to access KYC with an unsupported role, show an access denied message.
- **Failure/offline:** If KYC cannot reach SMS API (network error, backend downtime), show a non-technical error message with retry and limited offline behavior (e.g., cached last-known data if allowed by policy). Consider a generic “Service temporarily unavailable” state for full outages.
- **Other:** Handle expired sessions by redirecting the user to login with a clear message. For multi-child parents, handle the case where one child is removed from the school by updating the account switcher list cleanly.

## 7. MVP Scope

### In scope
- Single and multi-child access for parents and students tied to a specific school_id.
- Responsive navigation structure with Home feed, Transport, Diary, Account as top-level destinations (plus access to other modules).
- Instagram-like Home feed presenting school posts/events with a share button and no likes/comments.
- Gmail-like multi-child account switcher pattern for parents.
- Integration with SMS API for authentication, configuration, and data retrieval.

### Out of scope (backlog)
- Advanced personalization (per-user layout customization, themes beyond school branding).
- Cross-school parent accounts (e.g., children in multiple schools) in a single unified view.
- Offline mode with full data synchronization and conflict resolution.
- Full internationalization with multiple languages and right-to-left layout support.

### Success criteria for MVP
- At least one pilot school successfully onboards students and parents to KYC with verified usage of core modules (attendance, marks, fees, notifications) via the KYC navigation.
- At least 70% of surveyed parents report that it is easy to find key information (attendance, marks, fees, school updates) in KYC without contacting the school for help.
- Measurable reduction in paper-based notices and in-person visits for fee payment at pilot schools, compared to a pre-KYC baseline.

