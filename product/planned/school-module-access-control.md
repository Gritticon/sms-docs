---
title: School Module Access Control
type: requirement
project: platform
status: planned
last-updated: 2026-04-14
---

# School Module Access Control

## Overview

Each school onboarded on the Gritticon platform pays based on the modules they choose to subscribe to. This document specifies the feature that allows the Gritticon admin panel to configure exactly which SMS modules and submodules a school has access to, and how that configuration propagates across the platform — affecting staff in SMS-UI and parents/students in KYC.

This is a **subscription-level gate**, distinct from the existing staff role-based permissions (RBAC). These two systems work in layers:

```
Layer 1 — School Subscription Gate (this document):
  Gritticon admin configures which modules a school has purchased.
  No staff member in the school can access a module that hasn't been subscribed to,
  regardless of their role permissions.

Layer 2 — Staff Role Permissions (existing RBAC):
  Within the modules a school has access to, individual staff access is
  governed by their assigned role's permission matrix (180-permission system).
```

---

## Scope

This feature touches four parts of the platform:

| Component | What changes |
|---|---|
| **Admin Panel** | New UI to configure module access per school |
| **sms-api** | New data model, APIs to store and enforce module config |
| **SMS-UI** | Reads school module config; hides unsubscribed modules from navigation |
| **KYC** | Reads school module config; hides KYC modules whose SMS counterpart is not subscribed |

---

## Core Concepts

### Module vs Submodule

- A **module** is a top-level feature group (e.g. Fee Management).
- A **submodule** is a distinct feature within a module (e.g. Fee Structures, Fee Collection).
- A school can subscribe to an entire module (all submodules included) or to specific submodules only.
- If a school subscribes to at least one submodule, the parent module itself is considered accessible.

### Core Modules (Always On)

Some modules are fundamental to the platform and are always available to every school, regardless of their subscription. They cannot be deselected in the admin panel.

| Module | Reason |
|---|---|
| Dashboard | Primary landing screen; always required |
| School Management | School must be able to manage its own profile and settings |
| Staff Management → Staff Profiles | Core identity module |
| Class Management → Classes & Sections, Subjects | Foundation for all academic modules |
| Student Management → Student Profiles | Core student identity |
| Audit Logs | Compliance and accountability; non-negotiable |

> All other modules and submodules are optional and subscription-controlled.

### Gritticon Control vs School Control

Two separate layers of module visibility exist in the platform:

| Layer | Who controls it | Where it is configured | Purpose |
|---|---|---|---|
| **Subscription access** | Gritticon admin | Admin panel (this document) | Which modules the school has paid for |
| **KYC operational visibility** | School admin | SMS → School Management → KYC Settings | Of the modules available, which ones to show in KYC |

These are independent. A school may have subscribed to the Fees module but choose to hide it from KYC parents temporarily. Both conditions must be true for a KYC module to appear:
1. The school's subscription includes the parent SMS module.
2. The school admin has not hidden it via KYC Settings.

---

## Module Catalog

This is the complete list of modules and submodules that can be controlled via school module access. Core modules are marked.

### SMS-UI Modules

| Module | Module ID | Submodule | Submodule ID | Core |
|---|---|---|---|---|
| Dashboard | 1 | — | — | Yes |
| Communication Hub | 2 | Messages | 101 | No |
| | | Notifications | 102 | No |
| | | Announcements | 103 | No |
| Transport Management | 3 | Routes | 201 | No |
| | | Vehicles | 202 | No |
| | | Drivers | 203 | No |
| Student Management | 4 | Student Profiles | 301 | Yes |
| | | House Tag | 305 | No |
| | | Certificates | 306 | No |
| | | Documents | 307 | No |
| | | Achievements | 308 | No |
| | | Code of Conduct | 309 | No |
| | | Requests | 311 | No |
| Exam Management | 5 | Exams | 400 | No |
| | | Question Papers | 402 | No |
| | | Results | 403 | No |
| | | Report Cards | 404 | No |
| Staff Management | 6 | Staff Profiles | 501 | Yes |
| | | Staff Attendance | 504 | No |
| | | Payroll | 505 | No |
| | | Leave Management | 506 | No |
| | | Roles & Departments | 507 | Yes |
| Class Management | 7 | Classes & Sections | 601 | Yes |
| | | Subjects | 605 | Yes |
| | | Assignments | 603 | No |
| | | Class Attendance | 604 | No |
| Complaints | 8 | View Complaints | 701 | No |
| | | Resolve Complaints | 702 | No |
| School Management | 9 | School Info | 801 | Yes |
| | | School Settings | 802 | Yes |
| Schedule Management | 10 | Timetable | 602 | No |
| | | Teacher Allotment | 606 | No |
| | | Holidays | 607 | No |
| | | Substitute | 608 | No |
| Watchlist | 11 | — | — | No |
| Class Session | 12 | Class Progress | 902 | No |
| Audit Logs | 13 | Audit Logs | 1301 | Yes |
| Fee Management | 14 | Fee Structures | 1401 | No |
| | | Fee Collection | 1402 | No |

> Teacher Diary, Inventory Management, and Order Management are pending SMS modules. When built, they will be added to this catalog with their own IDs. The access control system must accommodate new modules without schema changes — a forward-compatible design is required.

### KYC Module → SMS Module Dependency Map

Each KYC module is unlocked only when its dependent SMS module is part of the school's subscription.

| KYC Module | Depends on SMS Module / Submodule |
|---|---|
| Home Feed | Communication Hub |
| Attendance | Student Management → Student Profiles |
| Marks / Exams | Exam Management |
| Timetable | Schedule Management → Timetable |
| Diary | Staff Management (Teacher Diary when built) |
| Fees | Fee Management |
| Orders | Order Management (when built) |
| Transport | Transport Management |
| Notifications | Communication Hub |
| Messages | Communication Hub |
| Complaints | Complaints |
| Documents | Student Management → Documents |
| Library | Inventory Management (when built) |
| Reminders | Fee Management (fees-based) OR always on |
| Student Profile | Student Management → Student Profiles |
| Account | Always on (core) |

> If the dependent SMS module is not subscribed, the KYC module is not shown in the KYC navigation. The route itself returns a "not available for your school" screen rather than a 404, so deep links don't break hard.

---

## Admin Panel Requirements

### Where it lives

Module access configuration is part of each school's record in the admin panel. It is accessible from the **School Detail** screen, as a new dedicated tab.

**Navigation path:**
```
Admin Panel → Schools → [School Detail] → Module Access tab
```

### Screen: Module Access

This screen allows a Gritticon admin to view and configure which modules and submodules are enabled for the selected school.

#### Access Rules

| Role | Can view | Can edit |
|---|---|---|
| Super Admin | Yes | Yes |
| Billing Manager | Yes | Yes |
| Support Agent | Yes | No |
| Viewer | Yes | No |

#### Layout

The screen displays a grouped list of all SMS modules. Each module is an expandable card showing its submodules.

**Module card (collapsed state):**
- Module name
- Toggle switch (on/off for the entire module)
- Badge: "X / Y submodules enabled" when partially enabled
- "Core" label for core modules (toggle is disabled and locked ON)
- Expand chevron

**Module card (expanded state):**
- Each submodule listed as a row with a toggle switch
- Submodule name
- "Core" label for core submodules (toggle locked ON)

**Module-level toggle behaviour:**
- Turning a module OFF deselects all non-core submodules and saves
- Turning a module ON enables all non-core submodules by default (admin can then deselect individual submodules after expanding)
- Core submodules remain ON regardless

**Visual hierarchy:**
- Core modules: shown first in the list, greyed out lock icon on the toggle
- Optional modules: shown below, full control
- Modules not yet built (pending): shown in a separate "Coming Soon" section, non-interactive, informational only

#### Save Behaviour

Changes are not auto-saved. A **Save Changes** button is shown in a sticky footer whenever there are unsaved changes. Navigating away with unsaved changes shows a confirmation dialog.

On save:
- The new module config is written to the database
- A Gritticon admin audit log entry is created: `{ action: module_access_updated, school_id, changed_by, previous_config, new_config, timestamp }`
- The school's JWT tokens are NOT invalidated immediately — module access is enforced at the API level on each request, so the next request the school staff makes will reflect the new config

#### Change History

Below the module access grid, a read-only **Change History** log shows the last 20 config changes for this school:

| Timestamp | Changed by | Summary |
|---|---|---|
| 2026-04-14 10:23 | Mahesh (Super Admin) | Enabled: Fee Management (Fee Structures, Fee Collection) |
| 2026-04-10 14:05 | Priya (Billing Manager) | Disabled: Transport Management |

---

### Onboarding Integration

When a new school is onboarded via the **Onboard New School** flow, a step is added to configure the initial module access.

**New Step 2.5 — Module Selection (inserted between Initial Configuration and Admin Account):**
- Uses the same module access grid UI, but in a simplified "select what they've paid for" framing
- A "Standard Package" preset can be applied with one click (defines a sensible default set)
- Individual modules/submodules can be adjusted from the preset
- If skipped, the school defaults to core modules only; modules can be configured later from the School Detail screen

---

## API Requirements (sms-api)

### Data Model

A new table `school_module_access` stores the per-school module configuration.

```
school_module_access
├── id              (int, PK)
├── school_id       (int, FK → schools.id)
├── module_id       (int)
├── submodule_id    (int, nullable — null means the entry covers the whole module)
├── is_enabled      (bool)
├── updated_at      (datetime)
└── updated_by      (int, FK → internal_employees.id)
```

**Key design decision:** Only store rows for optional modules. Core modules are never in this table; they are unconditionally available. This means: if no row exists for a `(school_id, module_id)` pair, it defaults to disabled (opt-in model).

**Submodule granularity:** A row with `submodule_id = null` and `is_enabled = true` means the entire module is on. Rows with `submodule_id` set override at the submodule level. When reading access, submodule-level rows take precedence over module-level rows.

### Admin Panel APIs

```
GET    /internal/schools/{id}/module-access
       → Returns full module config for the school
         { modules: [{ module_id, is_enabled, submodules: [{ submodule_id, is_enabled }] }] }

PUT    /internal/schools/{id}/module-access
       → Replaces the full module config for the school
         Body: same structure as GET response
         Auth: internal_employee JWT (Super Admin or Billing Manager)
         Effect: overwrites existing rows, logs the change to admin audit log
```

### School-Facing Access Check API

The school-facing JWT (staff login) needs a way to retrieve its own module config on login/startup.

```
GET    /schools/{school_id}/module-access
       → Auth: valid staff JWT for this school
       → Returns the list of enabled module and submodule IDs for the school
         { enabled_modules: [1, 4, 5, 6, 7, 9, 13], enabled_submodules: [301, 501, 601, 605, 801, 802, 1301] }
       → Core modules/submodules are always included in this response
       → This endpoint is called once at login; the result is cached client-side per session
```

### Runtime Enforcement

Beyond the client hiding modules from navigation, the API must enforce access at the route level.

**Middleware:** A new `check_module_access` middleware is added to all non-auth, non-core API routes. It:
1. Reads `school_id` from the JWT
2. Reads `module_id` (derived from the route path or a route-level annotation)
3. Checks `school_module_access` table for the school + module pair
4. If not enabled → returns `403 { detail: "This module is not available for your school." }`
5. Core module routes are exempt from this check

This server-side enforcement ensures that even if a client-side bug or direct API call bypasses the UI gate, the API remains the authoritative enforcer.

---

## SMS-UI Impact

### Startup / Login Flow

After a successful login, SMS-UI calls `GET /schools/{school_id}/module-access` and stores the result in a `SchoolModuleAccessService` (a singleton service, not controller).

This service exposes:
```dart
bool isModuleEnabled(int moduleId)
bool isSubmoduleEnabled(int moduleId, int submoduleId)
```

### Navigation Filtering

The left sidebar / navigation rail reads `SchoolModuleAccessService` to decide which modules to show. Disabled modules are not rendered at all — no "locked" state visible to staff, no upsell prompt. The module simply does not exist in their UI.

**Why no locked/upsell state in SMS-UI:** Staff are not the buyers. Showing them a "purchase this module" screen creates confusion and noise. They simply see what their school has. The business conversation happens between Gritticon and the school management.

### Direct URL Navigation

If a staff member navigates directly to a module URL that their school doesn't have access to (e.g. via a bookmark), the screen shows a "This module is not available for your school" message rather than a crash or unrelated error.

### No Change to RBAC

The existing role permission matrix (180 permissions) is unchanged. The module access check happens before the role permission check — if the module is not subscribed, RBAC is never evaluated.

---

## KYC Impact

### Startup

KYC also calls `GET /schools/{school_id}/module-access` at login (same endpoint, student/parent JWT). It stores the enabled list and uses it to filter the navigation.

### Navigation Filtering

Bottom navigation (mobile) and sidebar rail (tablet/web) items are filtered to only show KYC modules whose SMS dependency is enabled for the school.

**Dependency resolution example:**
- If `Communication Hub` (module 2) is not in `enabled_modules` → Home Feed, Notifications, Messages are hidden from KYC navigation
- If `Fee Management` (module 14) is not enabled → Fees is hidden
- If `Transport Management` (module 3) is not enabled → Transport is hidden

**Account** is always shown (it's core — logout, child switcher, settings).

### KYC Operational Settings Interaction

School Management → KYC Settings allows the school admin to show/hide KYC modules from parents. This is an additional layer on top of subscription access:

```
KYC module is visible if:
  (1) The school's subscription includes the parent SMS module
  AND
  (2) The school admin has not hidden it via KYC Settings
```

The subscription check (1) is enforced by the backend. The KYC Settings check (2) is an additional preference layer. If (1) is false, the KYC Settings toggle for that module should be grayed out and non-interactive in the School Management UI, with a tooltip: "Not available in your current plan."

---

## Edge Cases & Rules

| Scenario | Behaviour |
|---|---|
| School has no module config rows (new onboarding, no config set) | Only core modules are accessible. All optional modules are disabled. |
| Admin disables a module that a school's staff is currently using | On their next API request, the API returns 403. The UI will reflect the change on next reload. No data is deleted. |
| Admin enables a module that has never been used | The module appears in navigation. Existing data is untouched (modules may have been used before being disabled). |
| A new submodule is added to an existing module | New submodule defaults to disabled for all schools until explicitly enabled via admin panel. |
| A pending module (e.g. Inventory Management) is built and deployed | Its rows in `school_module_access` are inactive until Gritticon admin explicitly enables it per school. Existing schools do not get automatic access. |
| Core module is included in a PUT request body as disabled | Server ignores the disabled flag for core modules, treats them as enabled regardless. No error is returned — idempotent. |
| KYC Settings hides a module but the subscription is active | Module is hidden from KYC parents as the school admin intended. Module remains fully accessible to staff in SMS-UI. |
| School is SUSPENDED | Module access config is irrelevant — all logins are blocked at the auth level (existing behaviour). |
| School's module access is changed while staff are logged in | Existing sessions are not invalidated. The change is enforced at the next API call. No forced logout. |

---

## Impact on Existing Features

### School Management → KYC Settings

Currently, KYC Settings in SMS School Management allows the school admin to show/hide KYC modules. After this feature is built:

- The list of toggles in KYC Settings must be filtered to only show modules the school has subscribed to
- Modules not subscribed appear as greyed-out rows with a "Not in your plan" label and are non-interactive
- This prevents school admins from trying to toggle modules they don't have

### Admin Panel — School Detail Page

The existing School Detail page gains a new **Module Access** tab alongside the existing details, pricing, and activity information.

### School Onboarding

The existing 3-step onboarding form gains a **Module Selection** step (Step 2.5) as described above.

---

## Open Questions

| Question | Decision needed |
|---|---|
| Should there be predefined "plan tiers" (e.g. Basic, Standard, Premium) or fully custom per school? | Predefined tiers would make onboarding faster but reduce flexibility. Custom per school is fully flexible. This affects whether the admin panel shows a "plan selector" or a raw module checklist. |
| Should staff see a "not in your plan" message on restricted modules, or simply not see them at all? | The current proposal is to hide completely. If upsell messaging to school management (not staff) is needed, it could be surfaced elsewhere (e.g. admin panel usage dashboard). |
| Does disabling a module mid-cycle affect pricing? | Pricing reconciliation on mid-cycle changes is a billing concern — out of scope for this document. Needs definition in the pricing module. |
| Should KYC's Account page list the modules the student has access to? | Informational transparency vs UX simplicity. Currently proposed: no — KYC just hides unavailable items silently. |
| Should there be a "trial" mode per module — where a school can preview a module for N days before subscribing? | Future consideration. Not in scope for initial build. |

---

## Implementation Order

This feature should be built in the following sequence to ensure each piece is independently usable:

```
Step 1 — sms-api data model
  Create school_module_access table + migration
  Seed core module definitions as a constant (not DB rows)

Step 2 — Admin panel read endpoint + Module Access tab (read-only)
  GET /internal/schools/{id}/module-access
  Admin panel: School Detail → Module Access tab (view only, no editing yet)

Step 3 — Admin panel write endpoint + edit UI
  PUT /internal/schools/{id}/module-access
  Admin panel: enable editing, Save Changes button, change history log

Step 4 — Onboarding integration
  Add Module Selection step to the Onboard New School flow

Step 5 — SMS-UI enforcement
  GET /schools/{school_id}/module-access (school-facing)
  SchoolModuleAccessService in sms-ui
  Navigation filtering based on module access

Step 6 — KYC enforcement
  KYC reads same endpoint; filters navigation
  Updates KYC Settings in School Management to respect subscription

Step 7 — API-level middleware enforcement
  check_module_access middleware on sms-api routes
  Server-side 403 for unsubscribed modules
```

> Steps 1–4 can go live and be used by the Gritticon team to configure schools before SMS-UI and KYC enforcement is built. This means the system can be configured and verified from the admin side before it affects any school's live usage.
