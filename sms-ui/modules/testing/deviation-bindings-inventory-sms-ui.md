---
title: "Deviation: Bindings folder only covers audit log — sms-ui"
type: deviation
project: sms-ui
module: GetX bindings
last-updated: 2026-04-08
---


> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-03-21.

## Deviation details

- **File:** `sms-ui/lib/bindings/` (only `audit_log_binding.dart` present)
- **Violation type:** Pipeline check expected each major module to have a binding wiring controllers; most screens are not represented here.
- **Expected pattern:** `docs/sms-ui/architecture/modules-and-submodules.md` — modules map to navigation; bindings often used with GetX for module wiring.
- **Actual:** App uses presenters, GoRouter, and inline controller construction in views; only Audit Log has a dedicated binding file in this folder.

## Required action

- [ ] Confirm whether bindings are intentionally omitted elsewhere (presenter + router pattern).
- [ ] If not, add bindings per module or document the canonical wiring pattern in `docs/sms-ui/architecture/`.

## Reference

- Architecture doc: `docs/sms-ui/architecture/modules-and-submodules.md`
