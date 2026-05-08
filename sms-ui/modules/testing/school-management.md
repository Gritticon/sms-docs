---
title: "School Management — SMS-UI"
type: planning-stub
project: sms-ui
module: School Management
last-updated: 2026-03-23
---

> Auto-promoted to testing on 2026-03-23 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Feature is in the product roadmap (Module 9) but has no SMS-UI implementation specification.

## What this feature is

School Management is the settings and configuration centre for a school's SMS instance. It is used by the school's Super Admin to configure the platform for their school:
- Edit school profile (name, logo, address, contact, academic year)
- Configure which KYC modules are visible to parents/students
- Manage school-level settings (branding: primary colour for KYC, fee settings, etc.)
- Manage payment gateway configuration
- Academic year / term configuration
- Holiday calendar management (global holidays)
- Manage school-wide notifications defaults

The Gritticon Admin Panel handles school onboarding — this is the **within-school settings UI** for the school's own admin.

## What needs to be planned

- [ ] School profile page (edit name, logo, address, academic year, timezone)
- [ ] KYC module visibility toggles (which modules parents can see — fee, orders, transport, etc.)
- [ ] Primary colour / branding settings (drives KYC theme)
- [ ] Payment gateway configuration (credentials for online fee collection)
- [ ] Fee settings (late fee rules, grace periods, partial payment policy)
- [ ] Academic year / term management (dates, holidays, exam periods)
- [ ] Who has access to School Management? (Super Admin only? What permissions?)
- [ ] Relationship with Admin Panel — what can Gritticon override vs. school controls?

## Related docs

- Admin Panel spec (school onboarding): `/doc/admin-panel/modules/built/admin-panel-spec`
- Module registry: `/doc/sms-ui/architecture/modules-and-submodules`
- KYC overview (module visibility): `/doc/kyc/specs/built/kyc-overview-spec`
- Feature matrix: `/doc/product/feature-matrix`

## Dependencies

- SMS-API school settings endpoints (partially built — needs extension)
- KYC app reads school settings to control which modules show in navigation
- Payment gateway integration (needs separate planning)
