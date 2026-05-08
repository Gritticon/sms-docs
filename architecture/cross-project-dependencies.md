---
title: Cross-Project Dependencies
type: architecture
project: platform
last-updated: 2026-03-18
---

# Cross-Project Dependencies

This document defines the dependency relationships between all SMS platform projects. Anyone making a change in one project must consult this document to understand what else is affected.

---

## Dependency Map

```
                        ┌──────────────────┐
                        │    sms-api       │
                        │  (single source) │
                        └────────┬─────────┘
                                 │ API contract
               ┌─────────────────┼──────────────────┐
               ▼                                    ▼
        ┌─────────────┐                    ┌──────────────┐
        │   sms-ui    │                    │     kyc      │
        │ (staff side)│◄──── data flow ───►│(parent/      │
        └─────────────┘                    │ student side)│
               │                           └──────────────┘
               │ describes features                │
               │                                   │ describes features
               └──────────────┐  ┌─────────────────┘
                              ▼  ▼
                        ┌──────────┐
                        │ website  │
                        │(content) │
                        └──────────┘
```

---

## Dependency Types

### 1. API Contract Dependency

Both `sms-ui` and `kyc` consume `sms-api`. They share the same backend but access different endpoints and are scoped by their respective user types (staff vs parent/student).

| Consumer | Depends On | Nature |
|---|---|---|
| sms-ui | sms-api | All staff-facing endpoints |
| kyc | sms-api | All parent/student-facing endpoints |
| website | sms-api | None — no live data, static content only |

**Impact rule:** Any breaking change to a `sms-api` endpoint must be assessed against both `sms-ui` and `kyc`. New endpoints that expose data to parents/students require both a backend implementation in `sms-api` and a UI implementation in `kyc`.

---

### 2. Data Visibility Dependency

This is the most important cross-project relationship. Data created by staff in `sms-ui` is consumed by parents and students in `kyc` through `sms-api`. Some flows are bidirectional — parents/students can also create data in `kyc` that staff see in `sms-ui`.

**Unidirectional (sms-ui → kyc):** Staff writes, parent/student reads.
**Bidirectional:** Both sides create and read data for the same feature.

Full module-by-module mapping is in `architecture/data-flow.md`.

**Impact rule:** Any feature that produces data in `sms-ui` must be evaluated for whether it has a `kyc` counterpart. Conversely, any new `kyc` feature that allows parents/students to create data must have a corresponding view in `sms-ui` for staff to manage it.

---

### 3. Content Dependency

The `website` describes the features of both `sms-ui` and `kyc` in its marketing copy. This is not a code dependency — the website does not pull live data from `sms-api`. It is a content accuracy dependency.

| Website content about | Depends on accuracy of |
|---|---|
| Admin panel features | sms-ui built features |
| Parent/student app features | kyc built features |
| Platform feature list | Both sms-ui and kyc built features |

**Impact rule:** When a significant new feature ships in `sms-ui` or `kyc`, the website content team must be notified to update the relevant page copy. When a feature is removed or substantially changed, website copy must be updated. See `website/content/website-content.md` for the current copy.

---

## Impact Rules Summary

Use this table when making changes to understand what else needs attention.

| Change in | Must also check |
|---|---|
| `sms-api` — endpoint changed or removed | `sms-ui` (does it use this endpoint?), `kyc` (does it use this endpoint?) |
| `sms-api` — new endpoint added | `sms-ui` and/or `kyc` — is a UI needed for this endpoint? |
| `sms-api` — permission IDs changed | `sms-ui` permission registry, `sms-ui` docs, `sms-api` architecture docs |
| `sms-api` — data model changed | Any `sms-ui` or `kyc` views that display this data |
| `sms-ui` — new feature added | Data flow map: does it produce data a parent/student should see in `kyc`? Website: does website copy need updating? |
| `sms-ui` — feature removed | `kyc` counterpart (if any), website copy |
| `kyc` — new feature added | Data flow map: does it write data staff need to see in `sms-ui`? Website: does website copy need updating? |
| `kyc` — feature removed | `sms-ui` counterpart (if any), website copy |
| `website` — feature described that does not exist | Fix the copy — do not build the feature just because it is on the website |

---

## Cross-Project Feature Documentation Rule

When a feature spans multiple projects, each project documents its own side. The docs are linked to each other via a **Cross-Project** section.

**Example — Complaints feature:**
- `sms-ui/modules/built/complaints-module.md` — documents the staff-facing complaints management UI
- `kyc/specs/built/complaints-spec.md` — documents the parent-facing complaint submission UI
- Both docs contain a **Cross-Project** section pointing to the other

This keeps each doc focused on its own project while making the relationship explicit and discoverable.

---

## Bidirectional Features

These features have both a staff side (`sms-ui`) and a parent/student side (`kyc`) where both sides can create and read data. Extra care is needed when changing these — both sides must remain in sync.

| Feature | sms-ui side | kyc side |
|---|---|---|
| Complaints | Staff manage complaints raised on students | Parents raise complaints; teachers raise complaints |
| Requests | Staff manage and respond to requests | Parents raise requests |
| Messages | Staff send and receive messages | Parents/students send and receive messages |

---

## Related Documents

- `architecture/data-flow.md` — per-module data flow between projects
- `product/feature-matrix.md` — which features exist in which projects
- `process/documentation-system.md` — how cross-project docs are maintained
