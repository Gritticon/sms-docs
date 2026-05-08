---
title: "Deviation: Missing binding (code_of_conduct_add) — sms-ui"
type: deviation
project: sms-ui
module: code_of_conduct_add
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-04-08.

## Deviation details

- **File:** `sms-ui/lib/views/code_of_conduct_add_view.dart`
- **Violation type:** Missing submodule binding
- **Expected pattern:** Feature should have corresponding file in `lib/bindings/`.
- **Actual:** No matching binding detected.

## Required action

- [ ] Describe the specific fix needed
- [ ] Reference the architecture doc that defines the correct pattern
- [ ] Confirm whether this is intentional (tech debt accepted) or must be fixed before release

## Reference

- Architecture doc: `docs/sms-ui/architecture/modules-and-submodules.md`
