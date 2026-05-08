---
title: "Module not available — sms-ui"
type: planning-stub
project: sms-ui
module: gating
last-updated: null
---

## Status

**Needs spec** — Implementation detected in codebase during pipeline verify 2026-04-25; no `docs/sms-ui/` file references `module_not_available` / `module_not_available_view` (zero grep matches under `docs/sms-ui/`).

## What was found in code

- **File(s):** `sms-ui/lib/views/module_not_available_view.dart`
- **Type:** View file (likely placeholder when school module access disables a feature)
- **Detected endpoints/screens:** “Module disabled” or equivalent UX (copy and navigation to be confirmed from view code)

## What needs to be documented

- [ ] When this route is shown (router guard vs in-module check)
- [ ] Expected copy, recovery actions, analytics
- [ ] Link to `school_module_access` API and `docs/sms-api/.../school-module-access-api-surface.md` if product-facing

## Open questions

- Shared with other white-label gating or SMS-UI only?
