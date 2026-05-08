---
title: "Deviation: View without repository (timetable_preview) — sms-ui"
type: deviation
project: sms-ui
module: timetable_preview
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-04-08.

## Deviation details

- **File:** `sms-ui/lib/views/timetable_preview_view.dart`
- **Violation type:** No repository layer
- **Expected pattern:** Views should call API via repository layer in `lib/repositories/`.
- **Actual:** Network-related usage detected without matching repository.

## Required action

- [ ] Describe the specific fix needed
- [ ] Reference the architecture doc that defines the correct pattern
- [ ] Confirm whether this is intentional (tech debt accepted) or must be fixed before release

## Reference

- Architecture doc: `docs/sms-ui/architecture/modules-and-submodules.md`
