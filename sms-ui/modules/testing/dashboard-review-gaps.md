---
title: "Dashboard Module — Spec vs Implementation Gap Review"
type: review
project: sms-ui
module: Dashboard
last-updated: 2026-03-26
reviewed-spec: docs/sms-ui/modules/testing/dashboard-module.md
---

# Dashboard Module — Gap Review

**Last reviewed:** 2026-03-26
**Spec status:** `testing/` — awaiting QA sign-off
**Widget catalog revised:** 2026-03-26 — redesigned from 28 single-metric widgets to 21 high-signal role-scoped widgets
**Code reviewed:** `sms-ui/lib/views/dashboard_view.dart`, `widget_picker_view.dart`, `dashboard_presenter.dart`, `dashboard_repository.dart`, `dashboard_layout_model.dart`, `widget_catalog.dart`, `widgets/dashboard/dashboard_widget_renderer.dart`

---

## Overall Assessment

The **infrastructure layer is production-ready**. Edit mode, drag-to-reorder, widget picker, permission filtering, persistence, and unsaved-changes guard are all complete. The **critical gaps** are: (1) the widget catalog needs updating for the new 21-widget design, (2) none of the widgets render real data — all renderers are hardcoded placeholders, and (3) the renderer architecture needs rebuilding around the new widget types (donut stats, action lists, timelines, multi-stat cards).

| Layer | Status |
|---|---|
| Widget catalog (definitions, sizes, types) | Needs update — old 28-widget catalog, must be replaced with new 21-widget catalog |
| Permission filtering | ✓ Complete — module + sub-module level |
| Role-scoped data filtering | ✗ Not implemented — backend sends unfiltered data; frontend has no role-scope logic |
| Default layout generation | Needs update — priority widgets and caps need updating for new catalog |
| Edit mode (enter / save / discard / reset) | ✓ Complete |
| Widget picker (grouped, limit enforcement) | Needs update — grouping must change to Personal / Action Required / Monitoring |
| Drag-to-reorder | ✓ Complete |
| API persistence (GET/PUT /staff/me/dashboard) | ✓ Complete |
| Unsaved-changes guard (PopScope) | ✓ Complete |
| Widget data rendering (real API data) | ✗ Not implemented — all placeholders |
| Refresh button (re-fetches widget data) | ✗ No-op stub |
| Quick Links navigation | ✗ Placeholder chips only |
| Per-widget loading / error states | ✗ Not implemented |
| GetX bindings | ✗ None created |

---

## Gap 1 — Widget catalog must be rebuilt for new 21-widget design (Critical)

**Spec says:**
Widget catalog redesigned on 2026-03-26. 28 single-metric widgets replaced by 21 high-signal role-scoped widgets. New widget IDs, sizes, types (Action Required / Monitoring), and renderers.

**What was built:**
`widget_catalog.dart` contains the old 28-widget catalog with the original widget IDs (`student_count`, `staff_count`, `student_attendance_today`, etc.). These IDs no longer match the spec.

**What needs to happen:**
Rebuild `widget_catalog.dart` with the 21 new widget definitions. Key changes:
- New widget IDs (e.g. `student_attendance` replaces `student_count` + `student_attendance_today` + `student_attendance_trend`)
- Each widget entry now includes a `type` field: `ActionRequired` or `Monitoring`
- `WidgetCatalog.maxWidgetsFor(session)` needs updating for role-aware caps (admin: 20, standard: 15, restricted: 12)
- Priority widget map needs updating for default layout generation

---

## Gap 2 — Widget renderers are all placeholders, renderer architecture needs rebuilding (Critical)

**Spec says:**
> No new data endpoints are required for most widgets. Each widget fetches data from the existing module endpoints it belongs to.

**What was built:**
`dashboard_widget_renderer.dart` has five generic renderer types (`_StatCard`, `_ActivityFeed`, `_EventList`, `_ChartCard`, `_QuickLinks`). All render hardcoded content. The new widget catalog requires a different set of renderer types.

**New renderer types required:**

| Renderer | File | Used By |
|---|---|---|
| Donut Stat | `donut_stat_renderer.dart` | `student_attendance`, `staff_presence`, `my_attendance` |
| Action List | `action_list_renderer.dart` | `substitution_coverage`, `pending_results`, `open_issues`, `pending_approvals`, `my_pending_tasks` |
| Activity Feed | `activity_feed_renderer.dart` | `late_arrivals`, `inbox`, `student_welfare` |
| Event List | `event_list_renderer.dart` | `exam_schedule`, `upcoming_events` |
| Progress Stat | `progress_stat_renderer.dart` | `fee_collection` |
| Stat Card | `stat_card_renderer.dart` | `live_sessions`, `classes_coverage`, `library_status` |
| Status Breakdown | `status_breakdown_renderer.dart` | `transport_status` |
| Timeline | `timeline_renderer.dart` | `my_day` |
| Multi Stat | `multi_stat_renderer.dart` | `my_classes_health` |
| Quick Links | `quick_links_renderer.dart` | `quick_links` |

**Widget-to-endpoint mapping (updated for new catalog):**

| Widget ID | Data endpoint | Route file | Blocked? |
|---|---|---|---|
| `student_attendance` | `GET /students/attendance?date=today` + `GET /students/` | `attendance_buckets.py`, `students.py` | No |
| `staff_presence` | `GET /staff/` + `GET /staff/attendance?date=today` | `staff.py`, `staff_attendance.py` | No |
| `substitution_coverage` | `GET /substitute/?date=today` | `substitute.py` | No |
| `live_sessions` | `GET /class-sessions/?status=active` | `class_sessions.py` | No |
| `classes_coverage` | `GET /classes/` + `GET /sections/` + `GET /timetables/?date=today` | `classes.py`, `sections.py`, `timetables.py` | No |
| `exam_schedule` | `GET /exams/?upcoming=true` | `exams.py` | No |
| `pending_results` | `GET /exams/results?status=pending_entry` | `exams.py` | No |
| `open_issues` | `GET /support-tickets/?status=open` | `support_tickets.py` | No |
| `pending_approvals` | `GET /staff/leave-requests?status=pending` + fee/enrolment endpoints | TBD | Partially blocked |
| `fee_collection` | `GET /fees/collection?month=current` | TBD | Blocked — fee module not built |
| `late_arrivals` | `GET /students/attendance?type=late&date=today` | `attendance_buckets.py` | No (needs new query param) |
| `upcoming_events` | `GET /holidays/?upcoming=true` + `GET /events/?upcoming=true` | `holidays.py` | Partially (events endpoint TBD) |
| `inbox` | `GET /messages/unread` + `GET /announcements/unread` | TBD | Blocked — communication hub not built |
| `transport_status` | `GET /transport/routes?status=active` | TBD | Blocked — transport not built |
| `student_welfare` | `GET /students/attendance?consecutive_absent_min=3` | `attendance_buckets.py` (partial) | Partially blocked |
| `library_status` | `GET /library/books/overdue` + `GET /library/issues?date=today` | TBD | Blocked — library module not built |
| `quick_links` | No data fetch | — | No |
| `my_day` | `GET /timetables/?staff_id=me&date=today` | `timetables.py` | No |
| `my_classes_health` | `GET /class-sessions/?staff_id=me` + `GET /students/attendance?class_id=` | `class_sessions.py`, `attendance_buckets.py` | No |
| `my_attendance` | `GET /staff-attendance/?staff_id=me` | `staff_attendance.py` | No |
| `my_pending_tasks` | Aggregation from multiple sources scoped to user | TBD | Partially blocked |

**5 widgets fully blocked on unbuilt modules:** `inbox`, `transport_status`, `fee_collection`, `library_status` — show "Coming soon" empty state.

---

## Gap 3 — Role-scoped data not implemented (Critical)

**Spec says:**
> Widget data is role-scoped — the same widget shows different data depending on who is looking. Role-scoping is applied server-side.

**What was built:**
No role-scoping exists anywhere. `student_attendance` shows all students regardless of whether the viewer is a Class Teacher or a Principal. `open_issues` shows all complaints regardless of role.

**What needs to happen:**
Backend endpoints for role-scoped widgets need to filter based on the authenticated user's role and assignments (class assignments for teachers, department for department heads, etc.). This is a backend change, not a frontend change — the frontend should not need to send extra filter parameters.

---

## Gap 4 — Widget picker grouping is wrong (High)

**Spec says:**
> Widget picker groups widgets into three sections: Personal (top), Action Required (second), Monitoring (third).

**What was built:**
Widget picker groups by Personal and By Module. This does not reflect the new Action Required / Monitoring distinction that is central to the revised catalog design.

**What needs to happen:**
Update `widget_picker_view.dart` to group by: Personal → Action Required → Monitoring. Each administrative widget has a `type` field in the new catalog.

---

## Gap 5 — Refresh button is a no-op (High)

**Spec says:**
> A "Refresh" button re-fetches data for all widgets currently on the dashboard.

**What was built:**
`refreshData()` in `dashboard_presenter.dart` is an empty method stub.

**What needs to happen:**
Once real data fetching is wired, `refreshData()` triggers a re-fetch on all active widgets. Implementation depends on per-widget data state management.

---

## Gap 6 — Quick Links widget has no navigation (High)

**Spec says:**
> User selects module shortcuts; chips navigate to the module on tap. Config persisted in `config.module_slugs`.

**What was built:**
`_QuickLinks` renders six `ActionChip` widgets labelled "Module 1" through "Module 6". No config reading, no real module names, no navigation.

**What needs to happen:**
Read `config.module_slugs` from the layout entry, map slugs to module names and routes via the catalog, navigate on chip tap.

---

## Gap 7 — No per-widget loading or error states (Medium)

**What needs to happen:**
Each widget renderer must support three states:
- **Loading** — shimmer while API in-flight
- **Data** — renders actual content
- **Error** — "Could not load data" with retry button
- **Coming soon** — for widgets blocked on unbuilt APIs

---

## Gap 8 — GetX bindings not created (Low)

`dashboard_binding.dart` does not exist. The presenter is instantiated directly rather than via `Get.lazyPut`, deviating from the project's binding convention. Create the binding and register it in the router.

---

## Gap 9 — Widget picker not responsive (Low)

**Spec says:**
> Opens as a side panel (desktop) or bottom sheet (narrower viewports).

**What was built:**
Always opens as `EndDrawer` regardless of viewport width.

**What needs to happen:**
Add `MediaQuery` breakpoint check — `showModalBottomSheet` on narrow viewports, `EndDrawer` on desktop.

---

## Gap 10 — Permission revocation behaviour in `_sanitize()` (Low)

**Spec says:**
> Revoked widgets are hidden on render but their entry is preserved in the layout config so they reappear if permission is restored.

**What to verify:**
Check whether `_sanitize()` removes the widget ID from `_working` entirely (wrong) or merely excludes it from rendering (correct). If it removes and saves, the widget will not reappear on permission restoration.

---

## Recommended Completion Order

| Priority | Item | Effort |
|---|---|---|
| 1 | **Rebuild widget catalog** — new 21-widget definitions, IDs, types, sizes, priority map | Medium |
| 2 | **Grid layout + iOS-style cards** — replace column with staggered grid, card shell design | High |
| 3 | **Role-scoped data** — backend filtering for scoped widgets | High |
| 4 | **Wire real data for 16 buildable widgets** — implement renderer + API call for each | High |
| 5 | **Rebuild renderers** — donut stat, action list, activity feed, event list, timeline, multi-stat, progress bar | High |
| 6 | **Per-widget loading, error, coming-soon states** | Medium |
| 7 | **Update widget picker grouping** — Personal / Action Required / Monitoring | Low |
| 8 | **Quick Links — config reading + navigation** | Medium |
| 9 | **Refresh button** | Low (after data wiring) |
| 10 | **Add GetX binding** | Low |
| 11 | **Responsive widget picker** | Low |
| 12 | **Verify `_sanitize()` permission-revocation behaviour** | Low |
| 13 | **5 blocked widgets** — add "Coming soon" empty state | Low |
