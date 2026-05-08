---
title: "Mobile restriction gate — sms-ui"
type: planning-stub
project: sms-ui
module: Platform shell
last-updated: 2026-03-25
---

> Auto-promoted to testing on 2026-03-25 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Implementation detected in codebase during pipeline sync on 2026-03-23, but no specification exists.

## What was found in code

- **File(s):** `sms-ui/lib/views/mobile_restriction_view.dart`
- **Type:** View file (desktop-only messaging / layout gate)
- **Detected endpoints/screens:** UI that informs users the staff app is desktop-only; no API surface identified from this file alone.

## What needs to be documented

- [ ] When this view is shown (routing rules, breakpoints, feature flags)
- [ ] Relationship to navigation and login flow
- [ ] Whether this is temporary until responsive support ships

## Open questions

- Is this shown on all mobile browsers or only below a width threshold?
- Should this be listed in the official module registry?
