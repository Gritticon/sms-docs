---
title: "Transport Management — SMS-UI"
type: planning-stub
project: sms-ui
module: Transport Management
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Feature is in the product roadmap (Module 3) but has no SMS-UI implementation specification.

## What this feature is

Transport Management lets school admins configure and manage the school bus/vehicle fleet from the SMS staff portal:
- Define routes (stops, order, timing)
- Assign vehicles to routes
- Assign drivers to vehicles
- Assign students to routes/stops
- View student transport assignments

The parent/student side (route view, live tracking) is specced in KYC — this stub covers the **staff admin UI**.

## What needs to be planned

- [ ] Routes management screen (create route, add stops with sequence and timing)
- [ ] Vehicle management screen (add vehicle, registration, capacity)
- [ ] Driver management screen (add driver, contact, vehicle assignment)
- [ ] Student-to-route assignment (bulk assign by class/section or individual)
- [ ] Transport dashboard (overview: how many students per route, vehicle utilisation)
- [ ] How transport mode is reflected in student profile (walk/bus/private — already in Student Profile spec)
- [ ] Permission model (who can manage transport vs. view-only)
- [ ] Phase 2: Live GPS tracking admin view (deferred — track from this screen?)

## Related docs

- KYC side spec: `/doc/kyc/specs/built/transport-bus-tracking-spec`
- Module registry: `/doc/sms-ui/architecture/modules-and-submodules`
- Student profile (transport mode field): `/doc/sms-ui/modules/built/staff-profiles-how-it-works`
- Feature matrix: `/doc/product/feature-matrix`
- API needed: `sms-api/modules/needs-spec/transport-management-api`

## Dependencies

- SMS-API Transport Management endpoints (also needs-spec)
- Student Profile module (already built) — transport mode field lives here
- Phase 2: GPS/real-time bus tracking infrastructure
