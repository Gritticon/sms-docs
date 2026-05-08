---
title: "KYC settings — sms-ui"
type: planning-stub
project: sms-ui
module: KYC settings
last-updated: null
---

## Status

**Needs spec** — Implementation detected in codebase during pipeline verify 2026-04-25; no `docs/sms-ui/` file references `kyc_settings_view` (stem had zero grep matches under `docs/sms-ui/`).

## What was found in code

- **File(s):** `sms-ui/lib/views/kyc_settings_view.dart`
- **Type:** View file (staff web)
- **Detected endpoints/screens:** Settings UI for KYC / parent app integration (wire to `KycSettingsController` and related API in code review)

## What needs to be documented

- [ ] Feature specification and who can access (admin only vs module manager)
- [ ] API contract for any school-level KYC toggles
- [ ] Link to KYC `docs/kyc/` specs for parent-facing behavior

## Open questions

- Overlap with `kyc_settings` in `school_settings` / module gating — single source of truth doc?
