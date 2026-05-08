---
title: "Undocumented sms-ui screens — inventory"
type: planning-stub
project: sms-ui
module: (multiple)
last-updated: 2026-04-11
amended: "2026-04-11 (pipeline sync: re-scanned docs/sms-ui paths vs *_view.dart; prior long list superseded — most screens now have testing/* or built refs)"
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — 2026-04-11 pipeline sync: automated path scan (`docs/sms-ui/**/*.md` vs `*_view.dart` slug) found **no** matching doc path for the screens below. Other views now have `testing/` or `built/` specs or deviation stubs that reference the view path.

## What was found in code

- **File(s):** Listed below; each is a `lib/views/**/*_view.dart` screen with no `docs/sms-ui` markdown path containing the kebab-case slug of the view basename (and no `rg` hit on the view filename under `docs/sms-ui/`).
- **Type:** View files (staff web)
- **Detected screens (still no dedicated doc path match on 2026-04-11):**
  - `leave_settings_view.dart` / `LeaveSettingsView` — see `docs/sms-ui/modules/needs-spec/leave-settings-view.md`
  - `staff_attendance_settings_view.dart` / `StaffAttendanceSettingsView` — see `docs/sms-ui/modules/needs-spec/staff-attendance-settings-view.md`

> **Note:** Tighten coverage by adding module specs or merging these into leave / staff-attendance module docs once written.

## What needs to be documented

- [ ] Per-screen purpose, entry points, and permissions
- [ ] API contracts used (repository/presenter calls)
- [ ] Whether each should remain a standalone doc or fold into an existing module spec

## Open questions

- Should leave settings and staff attendance settings each get a full module spec or fold into existing leave / staff-attendance docs?
- Are both screens GA for all schools using staff attendance?
