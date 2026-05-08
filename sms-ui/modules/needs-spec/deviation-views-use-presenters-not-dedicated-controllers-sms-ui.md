---
title: "Deviation: Many views use presenters instead of per-view controllers — sms-ui"
type: deviation
project: sms-ui
module: State management
last-updated: null
---

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-03-21.

## Deviation details

- **File:** `sms-ui/lib/views/` (many `*_view.dart` files)
- **Violation type:** Automated check `*_view.dart` → matching `*_controller.dart` in `lib/controllers/` fails for most screens; the project uses `lib/presenters/` heavily (e.g. `StudentPresenter`, `ClassPresenter`).
- **Expected pattern:** Strict 1:1 view/controller naming is **not** how this codebase is structured.
- **Actual:** `lib/controllers/` has 20 files; `lib/presenters/` has 23 presenters; views often compose presenters and inline edit controllers.

## Required action

- [ ] Update the pipeline heuristic to treat presenters as the “logic layer” for screens where applicable.
- [ ] Document the presenter vs GetX controller rule in `docs/sms-ui/architecture/` if not already explicit.

## Reference

- Architecture doc: `docs/sms-ui/architecture/modules-and-submodules.md`
