---
title: Dashboard — Customisable Widget Dashboard
type: module
project: sms-ui
module: Dashboard
last-updated: 2026-03-26
---

> Auto-promoted to testing on 2026-03-25 — implementation detected in codebase. Widget catalog revised on 2026-03-26 — redesigned around daily staff roles and tasks. Awaiting QA sign-off before moving to built/.

# Dashboard — Customisable Widget Dashboard

## Table of Contents
1. [Overview](#overview)
2. [Widget Catalog](#widget-catalog)
3. [Widget Sizes](#widget-sizes)
4. [Grid & Layout System](#grid--layout-system)
5. [Permission Mapping](#permission-mapping)
6. [Default Layout Logic](#default-layout-logic)
7. [Customisation UX Flow](#customisation-ux-flow)
8. [Data Model](#data-model)
9. [Architecture in Flutter](#architecture-in-flutter)
10. [Decisions Log](#decisions-log)
11. [Open Questions](#open-questions)

---

## Overview

The dashboard is the landing screen for all roles after login. It is a fully customisable, permission-aware widget grid where each user can arrange widgets to suit their workflow.

**Key principles:**
- Each widget is a **situation report** — it shows multiple related data points together, not a single isolated number
- Widgets are separated by purpose: **Action Required** widgets surface things that need a decision or task, **Monitoring** widgets show ongoing state
- Widget data is **role-scoped** — the same widget shows different data depending on who is looking (e.g. a Class Teacher's Open Issues shows only complaints about their class; the Principal's shows all)
- Each user has their own layout, saved to the backend and consistent across sessions
- Widget availability is gated by the existing module permission system
- On first login (no saved layout), a recommended default set is generated based on the user's accessible modules and role
- Widgets are read-only summaries; any intended action navigates to the relevant module
- Widget cap is role-aware — admin roles have higher caps than restricted roles

---

## Widget Catalog

The catalog is split into two categories:

- **Administrative widgets** — show school-wide or aggregate data. Availability is gated by the existing module permission system. Data is role-scoped where relevant.
- **Personal widgets** — show data scoped to the logged-in staff member. Always available to any authenticated user regardless of module permissions.

---

### Administrative Widgets

Each widget is tagged as either **Monitoring** (passive, ongoing state) or **Action Required** (something needs to be done).

| Widget ID | Widget Name | Type | Module | Module ID | Submodule | Submodule ID | Visual Renderer | Accent |
|---|---|---|---|---|---|---|---|---|
| `student_attendance` | Student Attendance | Monitoring | Student Management | 4 | Student Profiles | 301 | Donut ring (% present) + stat breakdown | Blue `#3B82F6` |
| `staff_presence` | Staff Presence | Monitoring | Staff Management | 6 | Staff Profiles | 501 | Donut ring (% present) + stat breakdown | Indigo `#6366F1` |
| `substitution_coverage` | Substitution Coverage | Action Required | Schedule Management | 10 | Substitute | 608 | Action list with urgency indicator | Teal `#14B8A6` |
| `live_sessions` | Live Sessions | Monitoring | Class Session | 12 | Sessions | 901 | Stat card with linear progress bar | Orange `#F97316` |
| `classes_coverage` | Classes & Coverage | Monitoring | Class Management | 7 | Classes & Sections | 601 | Dual stat card (classes / sections) | Green `#22C55E` |
| `exam_schedule` | Exam Schedule | Monitoring | Exam Management | 5 | Exams | 400 | Event list with countdown | Amber `#F59E0B` |
| `pending_results` | Pending Results Entry | Action Required | Exam Management | 5 | Results | 403 | Action list with overdue indicator | Amber `#F59E0B` |
| `open_issues` | Open Issues | Action Required | Complaints | 8 | View Complaints | 701 | Action list with severity badges | Red `#EF4444` |
| `pending_approvals` | Pending Approvals | Action Required | _(cross-module)_ | — | — | — | Action list with urgency badges | Slate `#64748B` |
| `fee_collection` | Fee Collection | Monitoring | _(fee module — TBD)_ | — | — | — | Progress bar + stat breakdown | Emerald `#10B981` |
| `late_arrivals` | Late Arrivals | Monitoring | Student Management | 4 | Student Profiles | 301 | Activity feed with timestamps | Blue `#3B82F6` |
| `upcoming_events` | Upcoming Events | Monitoring | Schedule Management | 10 | Holidays | 607 | Event list with date labels | Teal `#14B8A6` |
| `inbox` | Inbox | Action Required | Communication Hub | 2 | Messages | 101 | Activity feed with unread badges | Purple `#A855F7` |
| `transport_status` | Transport Status | Monitoring | Transport Management | 3 | Routes | 201 | Status breakdown with route indicators | Cyan `#06B6D4` |
| `student_welfare` | Student Welfare | Monitoring | Student Management | 4 | Student Profiles | 301 | Activity feed with flag indicators | Pink `#EC4899` |
| `library_status` | Library Status | Monitoring | _(library module — TBD)_ | — | — | — | Dual stat card | Rose `#F43F5E` |
| `quick_links` | Quick Links | Utility | _(cross-module)_ | — | — | — | Configurable module shortcut chips | Slate `#64748B` |

---

#### What Each Widget Shows

**`student_attendance`**
> 310 / 342 present (90.6%) | Absent explained: 14 | Absent unexplained: 18 | Lowest attendance: Class 9B — 72%

**`staff_presence`**
> 31 / 40 present (77.5%) | On approved leave: 6 | Absent unnotified: 3 | 2 substitutes deployed today

**`substitution_coverage`**
> 3 substitutions needed | 2 assigned | 1 unassigned — Class 8A Period 3 (starts in 40 min)
> Absent teachers: Mr. Rajan (unnotified), Ms. Priya (approved leave)

**`live_sessions`**
> 12 / 18 classes in session | 3 starting within 30 min | 3 completed | 0 cancelled

**`classes_coverage`**
> 18 classes today | 17 / 18 covered | 6 sections | Timetable conflicts: 0

**`exam_schedule`**
> Next: Physics XII — Thu 27 Mar (2 days) | This week: 3 exams | Hall tickets issued: ✓

**`pending_results`**
> 3 exams awaiting result entry | Oldest: Maths 10B — 4 days overdue | 1 assigned to you

**`open_issues`**
> 4 open complaints | 2 unassigned | 1 escalated (>3 days) | Oldest: Cafeteria — 4 days unresolved

**`pending_approvals`**
> Leave requests: 3 pending | Fee waivers: 1 pending | Enrolments: 2 awaiting review | Urgent: 1 leave for tomorrow
> *(role-scoped: HR sees leave only; Accounts sees fee waivers only; Principal sees all)*

**`fee_collection`**
> ₹2.4L / ₹3.1L collected this month (77%) | Overdue: 48 students | > 30 days pending: 12 students

**`late_arrivals`**
> 6 students arrived late today | Last: Rohan Mehta — Class 9A — 9:42 am | Without reason: 2

**`upcoming_events`**
> Tomorrow: Parent-Teacher Meeting | Fri 28 Mar: Holi (holiday) | 2 more events this week

**`inbox`**
> 5 unread messages | 2 unread announcements | Latest: "Staff meeting at 3pm today" — Principal
> *(role-scoped: each staff member sees only their own messages and relevant announcements)*

**`transport_status`**
> 8 routes active | 6 on time | 1 delayed — Route 3 (15 min late) | 1 breakdown reported — Route 7 | 142 students in transit

**`student_welfare`**
> 4 students: 3+ consecutive absences | 2 open counselling cases | 1 flagged incident this week
> *(role-scoped: Class Teacher sees their class only; Counsellor and Principal see school-wide)*

**`library_status`**
> 12 books overdue | Longest overdue: 14 days (3 students) | 8 books issued today | 5 high-demand requests pending

**`quick_links`**
> User-configured module shortcut chips — up to 8 modules. Always available if the user has access to at least one module.

---

### Personal Widgets

Personal widgets show data scoped to the logged-in staff member. They are **always available in the widget picker** — no module permission check is applied.

| Widget ID | Widget Name | Type | Data Shown | Visual Renderer | Accent |
|---|---|---|---|---|---|
| `my_day` | My Day | Personal Schedule | The logged-in staff member's own schedule for today with next class highlighted | Horizontal timeline | Slate `#64748B` |
| `my_classes_health` | My Classes Health | Personal Monitoring | Per-class attendance, upcoming assessments, exam countdown for assigned classes | Multi-stat card (one row per class) | Slate `#64748B` |
| `my_attendance` | My Attendance | Personal Monitoring | Present days this month, leaves remaining, next approved leave | Donut ring + leave info | Slate `#64748B` |
| `my_pending_tasks` | My Pending Tasks | Personal Action Required | Personal action queue — results to enter, approvals to give, messages awaiting response | Action list | Slate `#64748B` |

---

#### What Each Personal Widget Shows

**`my_day`**
> Next: Maths 10A in 20 min — Room 204 | 3 classes remaining | Free period: 2:00–3:00 pm

**`my_classes_health`**
> Class 9A: 28 / 32 present | 2 assignments due this week | Exam in 4 days
> Class 10B: 30 / 34 present | Results pending entry

**`my_attendance`**
> Present 18 / 22 working days this month (81%) | Casual leaves: 2 remaining | Next leave: Mar 30

**`my_pending_tasks`**
> Results to enter: 2 exams | Approvals to give: 1 leave request | Messages to respond: 3
> *(aggregates action-required items scoped to the logged-in user's role and assignments)*

---

## Widget Sizes

Each widget has a fixed size defined in the catalog. Users can reorder widgets but cannot resize them.

The grid is column-based: **3 columns** on standard desktop, **2 columns** on narrower viewports. Widgets are rendered as iOS-style cards — rounded corners (16–20dp radius), module accent left-border stripe, module icon top-left, content fills the card body.

| Size | Grid Span (Columns × Rows) | Card Height | Best For |
|---|---|---|---|
| **Small** | 1 × 1 | ~140dp | Single stat with secondary context, dual numbers |
| **Medium** | 2 × 1 | ~180dp | Activity feeds, action lists, event lists, status breakdowns |
| **Wide** | 3 × 1 | ~180dp | Timelines, chip grids |
| **Large** | 2 × 2 | ~300dp | Donut rings with stat breakdown, multi-class health rows |

### Size Assignment per Widget

| Widget ID | Size |
|---|---|
| `student_attendance` | Large |
| `staff_presence` | Large |
| `substitution_coverage` | Medium |
| `live_sessions` | Small |
| `classes_coverage` | Small |
| `exam_schedule` | Medium |
| `pending_results` | Medium |
| `open_issues` | Medium |
| `pending_approvals` | Medium |
| `fee_collection` | Medium |
| `late_arrivals` | Medium |
| `upcoming_events` | Medium |
| `inbox` | Medium |
| `transport_status` | Medium |
| `student_welfare` | Medium |
| `library_status` | Small |
| `quick_links` | Wide |
| `my_day` | Wide |
| `my_classes_health` | Large |
| `my_attendance` | Small |
| `my_pending_tasks` | Medium |

---

## Grid & Layout System

- The grid is **reorderable** via drag-and-drop using `flutter_staggered_grid_view`
- Widgets snap to grid positions — no freeform placement
- Layout is stored as an **ordered list** (position = order in list), not as x/y coordinates — this keeps the layout responsive automatically
- Grid fills available width; widgets wrap naturally based on their column span
- **Widget cap is role-aware:**
  - Admin roles (access to all or most modules): **20 widgets**
  - Standard roles (limited module access): **15 widgets**
  - Restricted roles (1–2 modules, e.g. teacher-only): **12 widgets**
  - The cap is computed by `WidgetCatalog.maxWidgetsFor(session)` and surfaced clearly in the widget picker — when the cap is reached, the Add button is disabled with an inline message: _"You've reached the maximum of N widgets. Remove one to add another."_
- Quick Links' internal module chip selections do **not** count towards the widget cap

### Card Visual Design

Every widget card shares the same shell:
- **Background:** white (light mode) / dark surface (dark mode)
- **Border radius:** 16dp
- **Accent stripe:** 4dp left border in the module's accent colour
- **Module icon:** 20dp icon in the accent colour, top-left of the card header
- **Widget name:** small label (12sp muted) below the icon
- **Type badge:** Action Required widgets display a small amber dot indicator in the top-right of the header to distinguish them visually from Monitoring widgets
- **Content area:** fills the remaining card body; type-specific renderer (see Widget Catalog)
- **Edit mode overlay:** drag handle top-left, × remove button top-right; card dims to 70% opacity

---

## Permission Mapping

### Administrative Widgets

1. Each widget declares a `requiredModuleId` and optionally a `requiredSubmoduleId`
2. The existing `PermissionService` is used to check if the user has any permission for that module/submodule
3. The widget picker only shows administrative widgets the user has permission for
4. Default layout generation applies the same filter

**Role-scoped widgets:**
Several widgets show different data depending on the viewer's role. This is handled server-side — the backend uses the authenticated user's role and assignments to filter the response. The widget itself does not change; only the data changes.

| Widget ID | Role-scoping behaviour |
|---|---|
| `student_attendance` | Class Teacher sees their class only; Principal/VP sees school-wide |
| `open_issues` | Class Teacher sees complaints about their class; Principal sees all |
| `pending_approvals` | HR sees leave requests only; Accounts sees fee waivers only; Principal sees all |
| `inbox` | Each user sees only their own messages and announcements relevant to their role |
| `student_welfare` | Class Teacher sees their class only; Counsellor and Principal see school-wide |
| `pending_results` | Subject Teacher sees only their subject exams; Exam Controller sees all |
| `my_pending_tasks` | Always scoped to the logged-in user regardless of role |

**Edge case — permission revocation:**
If a user has an administrative widget on their saved layout and later loses the corresponding module permission, the widget is hidden on render but its entry is preserved in the layout config. If the permission is restored, the widget reappears in its original position automatically. `DashboardPresenter._sanitize()` must exclude rather than remove revoked widget IDs to support this.

### Personal Widgets

1. No permission check is applied — personal widgets are always shown in the widget picker
2. Personal widgets are included in the default layout generation for all users
3. If a staff member has no data for a personal widget (e.g. no timetable assigned, no subjects allocated), the widget renders a relevant empty state rather than being hidden

---

## Default Layout Logic

When a user has no saved layout, the system generates one using this logic:

1. Always start with the **personal widgets** — included for every user by default
2. Identify which modules the user has access to (via `PermissionService`)
3. For each accessible module, select the **highest-priority administrative widget** from the catalog for that module
4. Cap the default layout at **10 widgets total** (personal + administrative)
5. Always include **Quick Links** as the first widget if the user has access to more than 3 modules

### Priority Widget per Module

| Module | Priority Widget |
|---|---|
| Communication Hub | `inbox` |
| Transport Management | `transport_status` |
| Student Management | `student_attendance` |
| Exam Management | `exam_schedule` |
| Staff Management | `staff_presence` |
| Class Management | `classes_coverage` |
| Complaints | `open_issues` |
| Schedule Management | `substitution_coverage` |
| Class Session | `live_sessions` |
| Fee Management | `fee_collection` |
| Library | `library_status` |

### Example Default Layouts

**Principal (all modules):**
Quick Links → My Day → My Attendance → Student Attendance → Staff Presence → Substitution Coverage → Pending Approvals → Open Issues → Exam Schedule → Inbox

**Subject Teacher (no admin module permissions):**
My Day → My Attendance → My Pending Tasks → My Classes Health

**Class Teacher (Class Management + Student Management access):**
My Day → My Classes Health → Student Attendance → Exam Schedule → My Pending Tasks → My Attendance

**Receptionist (Communication Hub + Complaints):**
My Attendance → Inbox → Late Arrivals → Open Issues → Upcoming Events

**Exam Controller (Exam Management access):**
My Day → My Attendance → Exam Schedule → Pending Results Entry → My Pending Tasks

**Transport Manager (Transport Management access):**
My Attendance → Transport Status → Quick Links

**Librarian (Library access):**
My Attendance → Library Status → Inbox → Quick Links

---

## Customisation UX Flow

### Normal Mode
- The dashboard renders the widget grid
- A **"Customise"** button sits in the top-right of the dashboard header
- A **"Refresh"** button also sits in the top-right header area — tapping it re-fetches data for all widgets currently on the dashboard. Widget data is also fetched fresh on every page load.

### Edit Mode
Activated when the user taps "Customise".

- Each widget displays a **drag handle** (top-left corner) and a **remove button** (× top-right corner)
- The **Quick Links** widget additionally shows a **configure button** (pencil icon, bottom-right corner) — tapping it opens the Quick Links configuration sheet (see below)
- An **"+ Add Widget"** button appears — opens the widget picker
- A **"Done"** button replaces "Customise" — saves the current layout to the backend and exits edit mode
- A **"Reset to Default"** secondary action regenerates the system-recommended layout for the user
- If the user navigates away while in edit mode (clicks a sidebar link), a **Save or Discard prompt** is shown before navigating — the user must explicitly choose to save or discard their unsaved changes

### Quick Links Configuration Sheet
Opened by the pencil icon on the Quick Links widget card in edit mode.

- Opens as a **bottom sheet**
- Title: "Choose modules for Quick Links"
- Lists all modules the user has access to, each with a checkbox
- Maximum **8 modules** can be selected (enforced with inline message)
- Tapping "Save" stores the selection to `config.module_slugs` in the layout entry and updates the Quick Links card immediately
- On first use (no saved config), pre-populates with the user's top 6 accessible modules (same priority order as default layout algorithm)
- Changes are part of the wider edit-mode save — they are discarded if the user taps "Discard" on the main dashboard

### Widget Picker
- Opens as a side panel (on desktop) or bottom sheet (on narrower viewports)
- Widgets are grouped into three sections: **Personal** (always shown at the top), **Action Required** (administrative action widgets, second), and **Monitoring** (remaining administrative widgets, third)
- Each item shows: widget name, type badge (Action Required / Monitoring), size badge, brief one-line description
- Administrative widgets the user lacks permission for are **not shown**
- Personal widgets are **always shown** in the picker regardless of permissions
- Widgets already on the dashboard are shown with a **checkmark and are disabled** (cannot be added twice)
- Tapping an available widget **adds it immediately** to the end of the dashboard (live preview before saving)
- Picker can be closed without saving — changes only persist when "Done" is tapped

---

## Data Model

### Layout Storage (Backend)

The layout is stored as an ordered JSON array of widget entries on the staff profile (or a dedicated dashboard layout table).

```json
{
  "dashboard_layout": [
    { "widget_id": "quick_links", "position": 0, "config": { "module_slugs": ["student-management", "exams", "timetable"] } },
    { "widget_id": "student_attendance", "position": 1 },
    { "widget_id": "staff_presence", "position": 2 },
    { "widget_id": "pending_approvals", "position": 3 },
    { "widget_id": "my_day", "position": 4 }
  ]
}
```

**Field notes:**
- `widget_id` — stable string slug defined in the frontend widget catalog
- `position` — display order (0-indexed); the authoritative sort key
- `config` — optional object for widgets that require user configuration; currently only used by `quick_links` (`module_slugs: string[]`). Other widgets omit this field entirely.
- Widget size is **not stored** — it is fixed per widget type in the catalog
- The backend does not validate widget IDs against permissions or config values — that is purely a frontend concern
- Role-scoping is applied by the backend when serving widget data, not stored in the layout config

### API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/staff/me/dashboard` | Fetch the authenticated user's saved dashboard layout |
| `PUT` | `/staff/me/dashboard` | Save (overwrite) the authenticated user's dashboard layout |

**GET response:**
```json
{
  "success": true,
  "data": {
    "dashboard_layout": [
      { "widget_id": "quick_links", "position": 0, "config": { "module_slugs": ["student-management", "exams"] } },
      { "widget_id": "student_attendance", "position": 1 }
    ]
  }
}
```

If no layout has been saved yet, `dashboard_layout` returns `null` — the frontend then generates the default layout and saves it on first render.

**PUT request body:**
```json
{
  "dashboard_layout": [
    { "widget_id": "quick_links", "position": 0, "config": { "module_slugs": ["student-management", "exams"] } },
    { "widget_id": "student_attendance", "position": 1 }
  ]
}
```

### Widget Data Endpoints

No new data endpoints are required for most widgets. Each widget fetches data from the **existing module endpoints** it belongs to, filtered by the authenticated user's role and assignments server-side. See the gap review doc for the full widget-to-endpoint mapping.

Five widgets depend on backend modules not yet built:
- `inbox` — Communication Hub (not built)
- `transport_status` — Transport Management (not built)
- `fee_collection` — Fee module (not built, no module ID assigned)
- `library_status` — Library module (not built, no module ID assigned)
- `pending_approvals` — partially blocked (leave requests endpoint TBD)

These widgets render a "Coming soon" empty state until the relevant module is available.

---

## Architecture in Flutter

The dashboard follows the existing **MVP + GetX** pattern used throughout the app.

### Files

| Layer | File | Status | Purpose |
|---|---|---|---|
| Model | `lib/models/dashboard_layout_model.dart` | ✓ Built | Layout and widget entry data structures; includes optional `config` field |
| Model | `lib/models/widget_catalog.dart` | Needs update | Static catalog: all widget definitions, sizes, types (action/monitoring), module mappings, accent colours, priority order — update for new 21-widget catalog |
| Repository | `lib/repositories/dashboard_repository.dart` | ✓ Built | GET and PUT layout API calls |
| Presenter | `lib/presenters/dashboard_presenter.dart` | ✓ Built | Orchestrates layout load, permission filtering, default layout generation, save |
| View | `lib/views/dashboard_view.dart` | ✓ Built | Main dashboard grid, edit mode, drag-and-drop reorder |
| View | `lib/views/widget_picker_view.dart` | Needs update | Widget picker — update grouping: Personal / Action Required / Monitoring |
| Widget renderer | `lib/widgets/dashboard/dashboard_widget_renderer.dart` | Partial — placeholder data | Unified renderer; routes to type-specific sub-renderer; needs real data wiring |
| Renderer — Donut Stat | `lib/widgets/dashboard/renderers/donut_stat_renderer.dart` | Needs build | Donut ring + stat breakdown rows; used by student_attendance, staff_presence, my_attendance |
| Renderer — Action List | `lib/widgets/dashboard/renderers/action_list_renderer.dart` | Needs build | Ordered list of action items with urgency/severity badges; used by substitution_coverage, pending_results, open_issues, pending_approvals, my_pending_tasks |
| Renderer — Activity Feed | `lib/widgets/dashboard/renderers/activity_feed_renderer.dart` | Needs build | Scrollable list of items with timestamp or badge; used by late_arrivals, inbox, student_welfare |
| Renderer — Event List | `lib/widgets/dashboard/renderers/event_list_renderer.dart` | Needs build | Date-labelled upcoming event rows; used by exam_schedule, upcoming_events |
| Renderer — Progress Stat | `lib/widgets/dashboard/renderers/progress_stat_renderer.dart` | Needs build | Progress bar + supporting stat rows; used by fee_collection |
| Renderer — Stat Card | `lib/widgets/dashboard/renderers/stat_card_renderer.dart` | Needs build | Small single or dual stat; used by live_sessions, classes_coverage, library_status, my_attendance |
| Renderer — Status Breakdown | `lib/widgets/dashboard/renderers/status_breakdown_renderer.dart` | Needs build | Multi-status row display; used by transport_status |
| Renderer — Timeline | `lib/widgets/dashboard/renderers/timeline_renderer.dart` | Needs build | Horizontal timeline of today's schedule; used by my_day |
| Renderer — Multi Stat | `lib/widgets/dashboard/renderers/multi_stat_renderer.dart` | Needs build | One stat row per class/entity; used by my_classes_health |
| Renderer — Quick Links | `lib/widgets/dashboard/renderers/quick_links_renderer.dart` | Needs build | Module shortcut chips from `config.module_slugs`; navigates on tap |
| View — Quick Links config | `lib/views/quick_links_config_sheet.dart` | Needs build | Bottom sheet for selecting Quick Links module shortcuts in edit mode |
| Binding | `lib/bindings/dashboard_binding.dart` | Needs build | GetX lazy registration for `DashboardPresenter` and widget repositories |

### Dependencies
- `fl_chart: ^1.0.0` — ✓ already in `pubspec.yaml` — used for donut rings in attendance widgets
- `flutter_staggered_grid_view` — **add to `pubspec.yaml`** — required for the multi-column responsive grid layout with drag-to-reorder

### Services Used (Existing — No Changes)
- `PermissionService` — to filter the widget catalog by user permissions
- `SessionService` — to identify the current user and determine role-scoping
- `ApiService` — for the two dashboard layout endpoints and all widget data fetches

### Routing
- `home_view.dart` is replaced by `dashboard_view.dart`
- The existing router entry for the home/dashboard route is updated to point to `DashboardView`
- No new routes are needed; the widget picker is rendered as an overlay within the dashboard route

---

## Decisions Log

| # | Topic | Decision |
|---|---|---|
| 1 | **Backend** | New endpoints (`GET/PUT /staff/me/dashboard`) are implemented in `sms-api` |
| 2 | **Data refresh** | Widget data loads on page load and on manual tap of the Refresh button in the dashboard header. No auto-refresh interval. |
| 3 | **Charting library** | `fl_chart` is used. Donut ring for attendance percentage widgets (student, staff, my_attendance). |
| 4 | **Edit mode navigation away** | A Save or Discard prompt is shown if the user tries to navigate away while in edit mode. |
| 5 | **Quick Links widget** | User selects which modules appear (up to 8) via a configure sheet opened in edit mode. Selection is persisted in `config.module_slugs` inside the layout entry. Default pre-populates with top 6 accessible modules. |
| 6 | **Widget categories** | Two categories: Administrative (gated by module permissions) and Personal (always available, scoped to the logged-in user's own data). |
| 7 | **Grid layout** | `flutter_staggered_grid_view` replaces the single-column `ReorderableListView`. Grid is 3 columns on desktop, 2 on narrower viewports. Widget span is fixed per size tier (Small 1×1, Medium 2×1, Wide 3×1, Large 2×2). |
| 8 | **Card visual design** | Each widget renders as an iOS-style card: 16dp radius, module accent left-border stripe (4dp), module icon (20dp) top-left, type badge (amber dot) for Action Required widgets. |
| 9 | **Widget cap** | Cap is role-aware: admin → 20, standard → 15, restricted → 12. Computed by `WidgetCatalog.maxWidgetsFor(session)`. Quick Links chip selections do not count toward the cap. |
| 10 | **Widgets blocked on unbuilt APIs** | Five widgets (inbox, transport_status, fee_collection, library_status, pending_approvals partial) depend on backend modules not yet built. These widgets show a "Coming soon" empty state until the API is available. |
| 11 | **Widget catalog redesign** | Catalog redesigned on 2026-03-26 around the daily tasks of real school staff roles (Principal, Class Teacher, Exam Controller, HR, Receptionist, Librarian, Transport Manager, Counsellor, etc.) rather than around data modules. Reduced from 28 single-metric widgets to 21 high-signal multi-information widgets. |
| 12 | **Action Required vs Monitoring separation** | Widgets are explicitly tagged as either Action Required or Monitoring. Action Required widgets surface tasks that need a decision — substitution gaps, result entry overdue, complaints unassigned, approvals pending. These are separated from Monitoring widgets and grouped first in the widget picker. |
| 13 | **Role-scoped data** | Several widgets (student_attendance, open_issues, pending_approvals, inbox, student_welfare, pending_results, my_pending_tasks) show different data depending on the viewer's role and assignments. Role-scoping is applied server-side. The frontend sends no role filter — the backend derives it from the authenticated session. |
| 14 | **High-signal widget principle** | Every widget must show at least two related data points together. A widget that shows only a single number (e.g. "342 students") is not acceptable — it must show context (present/total, vs target, vs yesterday, breakdown by category). |
| 15 | **Fee and Library modules** | `fee_collection` and `library_status` widgets are included in the catalog but depend on modules with no assigned Module ID yet. These widgets are available in the picker for users with the relevant roles but render "Coming soon" until the modules are built and assigned IDs. |

---

## Open Questions

1. **Fee module** — No module ID is assigned for Fee Management. What is the module ID and submodule structure? Required before `fee_collection` widget can be permission-gated properly.
2. **Library module** — Is the Library module in scope for the SMS platform? If so, what is the module ID? `library_status` is blocked on this.
3. **Student Welfare data sources** — `student_welfare` shows "3+ consecutive absences" and "flagged incidents". Consecutive absence tracking requires the backend to compute streaks from attendance records — does this endpoint exist or need to be added? What constitutes a "flagged incident"?
4. **`pending_approvals` data sources** — Leave request approval is likely in Staff Management. Fee waiver approval is in Fee module. Enrolment approval is in Student Management. Does the backend need a unified approvals aggregation endpoint, or does the frontend call each module's pending endpoint separately?
5. **`my_pending_tasks` aggregation** — This widget aggregates action items from multiple sources (pending results for my subjects, pending approvals for my role, unread messages). Does the backend provide a unified endpoint for this, or does the frontend compose it from individual widget data?
6. **School Management module** — No widget is defined for this module (IDs 801, 802) as it is informational configuration. Confirm this is acceptable.
7. **Permission revocation in `_sanitize()`** — Verify that `DashboardPresenter._sanitize()` excludes rather than removes revoked widget IDs so they reappear automatically if permission is restored.
8. **Quick Links chip count** — Maximum of 8 module shortcuts per Quick Links widget is proposed. Confirm whether this is the right ceiling.
