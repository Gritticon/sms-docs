---
title: Dashboard — Customisable Widget Dashboard
type: module
project: sms-ui
module: Dashboard
last-updated: 2026-05-08
---

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
10. [Open Questions](#open-questions)

---

## Overview

The dashboard is the landing screen for all roles after login. It is a fully customisable, permission-aware widget grid where each user can arrange widgets to suit their workflow.

**Key principles:**
- Each user has their own layout, saved to the backend and consistent across sessions
- Widget availability is gated by the existing module permission system — no separate widget-level permission layer
- On first login (no saved layout), the system generates a recommended default set of widgets based on the user's accessible modules
- Widgets are read-only summaries; any intended action is a quick link that navigates to the relevant module
- A maximum of 12 widgets can be placed on the dashboard at once

---

## Widget Catalog

The catalog is split into two categories:

- **Administrative widgets** — show school-wide or aggregate data. Availability is gated by the existing module permission system.
- **Personal widgets** — show data scoped to the logged-in staff member. Always available to any authenticated user regardless of module permissions.

---

### Administrative Widgets

Each widget belongs to a module and optionally a submodule. Availability is determined by whether the user has any permission for that module/submodule via the existing `PermissionService`.

| Widget ID | Widget Name | Module | Module ID | Submodule | Submodule ID | Type |
|---|---|---|---|---|---|---|
| `student_count` | At a Glance: Students | Student Management | 4 | Student Profiles | 301 | Stat Card |
| `student_attendance_today` | Student Attendance Today | Student Management | 4 | Student Profiles | 301 | Stat Card |
| `student_recent_additions` | Recent Student Additions | Student Management | 4 | Student Profiles | 301 | Activity Feed |
| `student_attendance_trend` | Attendance Trend (Students) | Student Management | 4 | Student Profiles | 301 | Chart |
| `upcoming_exams` | Upcoming Exams | Exam Management | 5 | Exams | 400 | Event List |
| `recent_results` | Recent Results | Exam Management | 5 | Results | 403 | Activity Feed |
| `staff_count` | At a Glance: Staff | Staff Management | 6 | Staff Profiles | 501 | Stat Card |
| `staff_attendance_today` | Staff Attendance Today | Staff Management | 6 | Staff Attendance | 504 | Stat Card |
| `staff_attendance_trend` | Attendance Trend (Staff) | Staff Management | 6 | Staff Attendance | 504 | Chart |
| `pending_complaints` | Pending Complaints | Complaints | 8 | View Complaints | 701 | Stat Card |
| `recent_complaints` | Recent Complaints | Complaints | 8 | View Complaints | 701 | Activity Feed |
| `todays_timetable` | Today's Timetable | Schedule Management | 10 | Timetable | 602 | Calendar List |
| `upcoming_holidays` | Upcoming Holidays | Schedule Management | 10 | Holidays | 607 | Event List |
| `substitute_schedule` | Substitute Schedule | Schedule Management | 10 | Substitute | 608 | Activity Feed |
| `classes_overview` | Classes & Sections Overview | Class Management | 7 | Classes & Sections | 601 | Stat Card |
| `class_attendance_today` | Class Attendance Today | Class Management | 7 | Class Attendance | 604 | Stat Card |
| `upcoming_assignments` | Upcoming Assignments | Class Management | 7 | Assignments | 603 | Event List |
| `unread_messages` | Unread Messages | Communication Hub | 2 | Messages | 101 | Stat Card |
| `recent_announcements` | Recent Announcements | Communication Hub | 2 | Announcements | 103 | Activity Feed |
| `recent_notifications` | Recent Notifications | Communication Hub | 2 | Notifications | 102 | Activity Feed |
| `active_sessions` | Active Sessions | Class Session | 12 | Sessions | 901 | Stat Card |
| `class_progress` | Class Progress | Class Session | 12 | Class Progress | 902 | Chart |
| `transport_routes` | Transport Routes | Transport Management | 3 | Routes | 201 | Stat Card |

---

### Personal Widgets

Personal widgets show data scoped to the logged-in staff member. They are **always available in the widget picker** — no module permission check is applied. If the staff member has no relevant data (e.g. no timetable assigned), the widget renders an appropriate empty state.

| Widget ID | Widget Name | Data Shown | Type |
|---|---|---|---|
| `my_timetable_today` | My Timetable Today | The logged-in staff member's own schedule for today | Calendar List |
| `my_attendance` | My Attendance | The logged-in staff member's own attendance summary | Stat Card |
| `my_classes_today` | My Classes Today | The classes the logged-in staff member is teaching today | Event List |
| `my_subjects` | My Subjects | The subjects assigned to the logged-in staff member | Activity Feed |

---

## Widget Sizes

Each widget has a fixed size defined in the catalog. Users can reorder widgets but cannot resize them.

The grid is always **3 columns** regardless of viewport width.

| Size | Grid Span (Columns × Rows) | Best For |
|---|---|---|
| **Small** | 1 × 1 | Stat cards, single-number summaries |
| **Medium** | 2 × 1 | Activity feeds, event lists |
| **Wide** | 3 × 1 | Charts, timetable, quick links |
| **Large** | 2 × 2 | Charts with legend, multi-item feeds |

### Size Assignment per Widget

| Widget ID | Size |
|---|---|
| `student_count` | Small |
| `student_attendance_today` | Small |
| `student_recent_additions` | Medium |
| `student_attendance_trend` | Large |
| `upcoming_exams` | Medium |
| `recent_results` | Medium |
| `staff_count` | Small |
| `staff_attendance_today` | Small |
| `staff_attendance_trend` | Large |
| `pending_complaints` | Small |
| `recent_complaints` | Medium |
| `todays_timetable` | Wide |
| `upcoming_holidays` | Medium |
| `substitute_schedule` | Medium |
| `classes_overview` | Small |
| `class_attendance_today` | Small |
| `upcoming_assignments` | Medium |
| `unread_messages` | Small |
| `recent_announcements` | Medium |
| `recent_notifications` | Medium |
| `active_sessions` | Small |
| `class_progress` | Large |
| `transport_routes` | Small |
| `my_timetable_today` | Wide |
| `my_attendance` | Small |
| `my_classes_today` | Medium |
| `my_subjects` | Medium |

---

## Grid & Layout System

- The grid is **reorderable** via select-and-place (tap a widget to select it, tap another to swap positions)
- Widgets snap to grid positions — no freeform placement
- Layout is stored as an **ordered list** (position = order in list), not as x/y coordinates — this keeps the layout responsive automatically
- Grid fills available width; widgets wrap naturally based on their column span
- Maximum **12 widgets** allowed on the dashboard at once

---

## Permission Mapping

The two widget categories are treated differently.

### Administrative Widgets
1. Each widget declares a `requiredModuleId` and optionally a `requiredSubmoduleId`
2. The existing `PermissionService` is used to check if the user has any permission for that module/submodule
3. The widget picker only shows administrative widgets the user has permission for
4. Default layout generation applies the same filter

**Edge case — permission revocation:**
If a user has an administrative widget on their saved layout and later loses the corresponding module permission, the widget is hidden on render but its entry is preserved in the layout config. If the permission is restored, the widget reappears in its original position automatically.

### Personal Widgets
1. No permission check is applied — personal widgets are always shown in the widget picker
2. Personal widgets are included in the default layout generation for all users
3. If a staff member has no data for a personal widget (e.g. no timetable assigned, no subjects allocated), the widget renders a relevant empty state rather than being hidden
4. Permission revocation has no effect on personal widgets

---

## Default Layout Logic

When a user has no saved layout, the system generates one using this logic:

1. Always start with the **personal widgets** — these are included for every user by default
2. Identify which modules the user has access to (via `PermissionService`)
3. For each accessible module, select the **highest-priority administrative widget** from the catalog for that module (priority is predefined in the catalog)
4. Cap the default layout at **10 widgets total** (personal + administrative)

### Priority Widget per Module

| Module | Priority Widget |
|---|---|
| Communication Hub | `recent_announcements` |
| Transport Management | `transport_routes` |
| Student Management | `student_count` |
| Exam Management | `upcoming_exams` |
| Staff Management | `staff_count` |
| Class Management | `class_attendance_today` |
| Complaints | `pending_complaints` |
| School Management | _(no widget — informational only)_ |
| Schedule Management | `todays_timetable` |
| Class Session | `active_sessions` |

### Example Default Layouts

**Admin (all modules):**
My Timetable Today → My Attendance → At a Glance: Students → At a Glance: Staff → Pending Complaints → Upcoming Exams → Staff Attendance Today → Today's Timetable → Recent Announcements

**Teacher (no admin module permissions):**
My Timetable Today → My Attendance → My Classes Today → My Subjects

**Teacher (Class Management + Student Management access):**
My Timetable Today → My Classes Today → My Subjects → My Attendance → Class Attendance Today → Upcoming Assignments → Student Attendance Today → Upcoming Exams

**Receptionist (Communication Hub + Complaints):**
My Attendance → Recent Announcements → Unread Messages → Pending Complaints → Recent Complaints

---

## Customisation UX Flow

### Normal Mode
- The dashboard renders the widget grid
- A **"Customise"** button sits in the top-right of the dashboard header
- A **"Refresh"** button also sits in the top-right header area — tapping it re-fetches data for all widgets currently on the dashboard. Widget data is also fetched fresh on every page load.

### Edit Mode
Activated when the user taps "Customise".

- Each widget shows a **remove button** (× top-right corner)
- **Reordering** uses select-and-place:
  - Tap a widget → it highlights with a primary-colour border (selected)
  - Tap another widget of the **same size** → the two widgets swap positions; selection clears
  - Tap another widget of a **different size** → error shown; selection preserved so user can try another target
  - Tap the same widget again → deselects
  - Tap an empty gap slot → selected widget is placed into that gap if it fits (span check); error message if it doesn't fit; selection preserved so user can try another slot
- Empty gap slots in the grid are shown as dashed/outlined placeholders, sized to the available columns
- An **"+ Add Widget"** button appears — opens the widget picker; newly added widgets auto-place in the first fitting gap, or append to a new row if no gap fits
- A **"Done"** button replaces "Customise" — saves the current layout to the backend and exits edit mode
- A **"Reset to Default"** secondary action regenerates the system-recommended layout for the user
- If the user navigates away while in edit mode (clicks a sidebar link), a **Save or Discard prompt** is shown before navigating — the user must explicitly choose to save or discard their unsaved changes

### Widget Picker
- Opens as a side panel (on desktop) or bottom sheet (on narrower viewports)
- Widgets are grouped into two sections: **Personal** (always shown at the top) and **By Module** (administrative widgets grouped by their module)
- Each item shows: widget name, size badge, brief one-line description
- Administrative widgets the user lacks permission for are **not shown**
- Personal widgets are **always shown** in the picker regardless of permissions
- Widgets already on the dashboard are shown with a **checkmark and are disabled** (cannot be added twice)
- Tapping an available widget **adds it immediately** — placed in the first fitting gap in the current layout, or appended as a new row if no gap fits (live preview before saving)
- Picker can be closed without saving — changes only persist when "Done" is tapped

---

## Data Model

### Layout Storage (Backend)

A new field on the staff profile (or a dedicated dashboard layout table). The layout is stored as an ordered JSON array of widget entries.

```json
{
  "dashboard_layout": [
    { "widget_id": "quick_links", "position": 0 },
    { "widget_id": "student_count", "position": 1 },
    { "widget_id": "staff_attendance_today", "position": 2 },
    { "widget_id": "pending_complaints", "position": 3 }
  ]
}
```

**Field notes:**
- `widget_id` — stable string slug defined in the frontend widget catalog
- `position` — display order (0-indexed); the authoritative sort key
- Widget size is **not stored** — it is fixed per widget type in the catalog
- The backend does not validate widget IDs against permissions — that is purely a frontend concern

### New API Endpoints Required

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
      { "widget_id": "quick_links", "position": 0 },
      { "widget_id": "student_count", "position": 1 }
    ]
  }
}
```

If no layout has been saved yet, `dashboard_layout` returns `null` or an empty array — the frontend then generates the default layout and saves it on first render.

**PUT request body:**
```json
{
  "dashboard_layout": [
    { "widget_id": "quick_links", "position": 0 },
    { "widget_id": "student_count", "position": 1 }
  ]
}
```

### Widget Data Endpoints

No new data endpoints are required. Each widget fetches data from the **existing module endpoints** it belongs to. Widget implementations are responsible for knowing which endpoint to call and which fields to display.

---

## Architecture in Flutter

The dashboard follows the existing **MVP + GetX** pattern used throughout the app.

### New Files

| Layer | File | Purpose |
|---|---|---|
| Model | `lib/models/dashboard_layout_model.dart` | Layout and widget entry data structures |
| Model | `lib/models/widget_catalog.dart` | Static catalog: all widget definitions, sizes, module mappings, priority order |
| Repository | `lib/repositories/dashboard_repository.dart` | GET and PUT layout API calls |
| Presenter | `lib/presenters/dashboard_presenter.dart` | Orchestrates layout load, permission filtering, default layout generation, save |
| View | `lib/views/dashboard_view.dart` | Main dashboard grid, edit mode, select-and-place reorder, gap detection |
| View | `lib/views/widget_picker_view.dart` | Widget picker panel/sheet |
| Widgets (base) | `lib/widgets/dashboard/stat_card_widget.dart` | Reusable stat card layout |
| Widgets (base) | `lib/widgets/dashboard/activity_feed_widget.dart` | Reusable activity feed layout |
| Widgets (base) | `lib/widgets/dashboard/event_list_widget.dart` | Reusable event/upcoming list layout |
| Widgets (base) | `lib/widgets/dashboard/chart_widget.dart` | Reusable chart layout |
| Widgets (individual) | `lib/widgets/dashboard/widgets/student_count_widget.dart` | Example individual widget |
| Widgets (individual) | `lib/widgets/dashboard/widgets/...` | One file per widget in the catalog |

### New Dependencies
- `fl_chart` — to be added to `pubspec.yaml` for chart-type widgets (`student_attendance_trend`, `staff_attendance_trend`, `class_progress`)

### Services Used (Existing — No Changes)
- `PermissionService` — to filter the widget catalog by user permissions
- `SessionService` — to identify the current user
- `ApiService` — for the two new dashboard endpoints

### Routing
- `home_view.dart` is replaced by `dashboard_view.dart`
- The existing router entry for the home/dashboard route is updated to point to `DashboardView`
- No new routes are needed; the widget picker is rendered as an overlay within the dashboard route

---

## Decisions Log

Decisions made during planning, recorded here for reference.

| # | Topic | Decision |
|---|---|---|
| 1 | **Backend** | New endpoints (`GET/PUT /staff/me/dashboard`) are implemented in `sms-api` |
| 2 | **Data refresh** | Widget data loads on page load and on manual tap of the Refresh button in the dashboard header. No auto-refresh interval. |
| 3 | **Charting library** | A charting library will be added to the project. `fl_chart` is the recommended choice — it is the most widely adopted open-source Flutter charting package and has no licensing restrictions. |
| 4 | **Edit mode navigation away** | A Save or Discard prompt is shown if the user tries to navigate away while in edit mode. |
| 5 | **Reorder interaction** | Select-and-place replaces drag-and-drop. Tap to select, tap target to move. Gap slots validated for span fit before placement. |
| 6 | **Widget categories** | Two categories: Administrative (gated by module permissions) and Personal (always available, scoped to the logged-in user's own data). Personal widgets do not follow the permission system. |

---

## Open Questions

1. **School Management module** — No widget is defined for this module (IDs 801, 802) as it is informational configuration rather than actionable summary data. Confirm this is acceptable or whether a widget is needed (e.g., a school info card).
