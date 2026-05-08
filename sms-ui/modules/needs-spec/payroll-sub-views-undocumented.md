---
title: "Payroll sub-screens — sms-ui (needs spec)"
type: planning-stub
project: sms-ui
module: Payroll
last-updated: null
---

## Status

**Needs spec** — `payroll.md` and related testing docs do not name these view files. Pipeline verify on 2026-04-25 found **zero** matches in `docs/sms-ui/` for each stem below (separate from generic “payroll” keyword hits).

## What was found in code

- **Type:** Flutter `*_view.dart` under `sms-ui/lib/views/`
- **Files (no prior doc cross-reference):**
  - `payroll_assign_templates_view.dart`
  - `payroll_draft_detail_view.dart`
  - `payroll_earnings_masters_view.dart`
  - `payroll_finalized_view.dart`
  - `payroll_home_view.dart`
  - `payroll_settings_config_view.dart`
  - `payroll_templates_view.dart`

## What needs to be documented

- [ ] How each sub-screen maps to the payroll user journey and `PayrollRepository` / workspace APIs
- [ ] Or merge into a single `payroll-workflow` spec with explicit per-view file paths in a table
- [ ] Permission / RBAC per sub-route

## Open questions

- Should any of these be considered internal to `payroll-workspace` only (not separate product modules)?
