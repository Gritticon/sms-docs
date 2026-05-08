---
title: "Deviation: View without controller (requests) — sms-ui"
type: deviation
project: sms-ui
module: requests
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-04-08.

## Deviation details

- **File:** `sms-ui/lib/views/requests_view.dart`
- **Violation type:** View without controller
- **Expected pattern:** Views should be backed by GetX controller in `lib/controllers/`.
- **Actual:** No matching controller file detected.

## Required action

- [ ] Describe the specific fix needed
- [ ] Reference the architecture doc that defines the correct pattern
- [ ] Confirm whether this is intentional (tech debt accepted) or must be fixed before release

## Reference

- Architecture doc: `docs/sms-ui/architecture/modules-and-submodules.md`
