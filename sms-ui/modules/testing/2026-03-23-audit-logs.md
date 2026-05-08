---
title: Audit Logs — UI testing notes
type: testing
project: sms-ui
module: Audit Logs
last-updated: 2026-03-23
---

# Audit Logs — testing review (2026-03-23)

Short review of the Audit Log screen after manual testing. Items below are gaps or bugs relative to expected behaviour; see `docs/sms-ui/modules/built/audit-log-module.md` for the target spec.

---

## Findings

| # | Area | Issue | Notes / expected |
|---|------|--------|------------------|
| 1 | Filters | No submodule filter (or selection does not narrow results) | Spec: optional `submodule_id` filter combinable with module; list should not always load “everything” when submodule scoping is intended. |
| 2 | List / API | Staff name for the actor is missing or not shown | Spec: **Performed by** column should show staff name (and detail drawer: name + role). Confirm API returns display fields and UI binds them. |
| 3 | Data | Payload not stored and/or not returned for detail view | Spec: detail drawer shows a formatted payload (key–value); empty → “No payload recorded.” May need backend persistence/exposure + UI rendering. |
| 4 | Theme | Dark mode: primary (or filled) button label contrast is wrong | Labels should read clearly on dark surfaces (e.g. use `onPrimary` / theme tokens so text is not the same as surrounding theme text). |
| 5 | UI consistency | Detail drawer layout/pattern differs from other SMS drawers | Align padding, header, actions, and typography with shared drawer patterns used elsewhere in sms-ui. |
| 6 | Detail drawer | Module and submodule labels/context not clear or incomplete | Spec: show human-readable **Module** and **Submodule** (when applicable) in the drawer, consistent with list and API. |

---

## Suggested follow-up

1. **Backend:** Confirm audit list/detail responses include `submodule`, actor display name, and payload when recorded; align with `GET /audit-logs/` contract in the built module doc.
2. **Flutter:** Wire submodule filter if API supports it; fix theme tokens for buttons in dark mode; refactor detail drawer to shared drawer + module/submodule fields.
3. **Retest:** After changes, re-run filters (module + submodule + staff + date), dark mode, and drawer parity with another module’s drawer.
