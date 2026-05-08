---
title: "Progress Summary View — sms-ui"
type: planning-stub
project: sms-ui
module: Class Session
last-updated: 2026-03-25
---

> Auto-promoted to testing on 2026-03-25 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Implementation detected in codebase during pipeline sync verification on 2026-03-23, but no specification exists. This view was missed by the original 2026-03-23 pipeline scan.

## What was found in code

- **File(s):** `sms-ui/lib/views/progress_summary_view.dart`
- **Type:** View file (session progress summary screen)
- **Detected details:**
  - Uses `SessionLogsRepository` to fetch session-level progress data
  - Uses `ProgressSummaryModel` data model
  - Has parameter `onlyVisibleToStudents` (bool, default `false`) — enabling dual use in staff-facing and student-facing contexts
  - Integrates with `PermissionWrapper`, `GlobalService`, `LoadingService`
  - Related to Module 12 (Class Session) based on repository dependency

## What needs to be documented

- [ ] What data is shown (per-session progress metrics, aggregation, time range)
- [ ] Who uses this view and from where it is navigated to
- [ ] Behaviour of `onlyVisibleToStudents` flag — which routes pass `true` vs `false`
- [ ] Integration with Class Session module (Module 12, Submodule 901/902)
- [ ] Whether this overlaps with `session_logs_view.dart` and how they differ

## Open questions

- Is this view reachable from the current navigation in `navigation_view.dart`?
- Is the `onlyVisibleToStudents` path used in any KYC-facing flow, or purely internal to sms-ui?
- Should this be registered as a dedicated submodule in `modules-and-submodules.md`?
