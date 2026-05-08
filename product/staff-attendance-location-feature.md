---
title: Staff Attendance — Multi-Source Attendance System
type: product-feature
status: planning
last-updated: 2026-03-29
---

# Staff Attendance — Multi-Source Attendance System

## Overview

Staff attendance is an **event-driven, multi-source system**. Multiple sources (geofence, biometric, face scan, manual) feed attendance events into a unified record per staff per day. The system processes these events and determines the final attendance status automatically.

**Geofence is the primary source.** Manual is a fallback — used when staff have not installed the app, or to correct a wrong record. Other sources (biometric, face scan) can be integrated later without architectural changes.

This is a widely adopted model across Indian organisations. Staff install the SMS-UI Flutter Shell on their personal phones. Geofence detection runs in the background.

Delivered through the **SMS-UI Flutter Shell** — a native Flutter app wrapping the sms-ui web app. The shell handles all native device capabilities (GPS, background services, permissions). The web UI handles display and API calls.

---

## Mental Model

> The admin's job is **not** to mark attendance. The system marks attendance. The admin's job is to **review and correct exceptions.**

| Old assumption (manual-first) | Correct assumption (geofence-first) |
|---|---|
| Admin marks attendance daily | System marks attendance automatically |
| Manual entry is the primary action | Manual entry is an exception/correction |
| Attendance screen = input form | Attendance screen = monitoring dashboard |

---

## Source Priority

| Source | Status | When Used |
|---|---|---|
| Geofence | Primary | Staff has app installed and running |
| Manual | Fallback | App not installed, geofence failed, admin correction |
| Biometric | Planned | Future hardware integration |
| Face Scan | Planned | Future hardware integration |

All sources write to the same `attendance_events` table. The processing layer resolves events → final attendance record. Admin never needs to think about which source marked the attendance.

---

## Onboarding Transition

- Before app is installed: admin uses manual marking as usual
- After app is installed: geofence takes over; manual becomes exception-only
- No hard cutover — the two modes coexist naturally

---

## Architecture

```
Attendance Sources
    ├── Geofence (primary)
    │     └── Flutter SMS-UI Shell (background service)
    │           ├── ENTER school boundary → POST /attendance/event {source: geofence, type: clock_in}
    │           └── EXIT school boundary (outside break window) → POST /attendance/event {source: geofence, type: clock_out}
    │
    ├── Manual (fallback)
    │     └── Admin via sms-ui web → POST /attendance/event {source: manual, type: clock_in/out or status}
    │
    ├── Biometric (planned)
    │     └── Hardware integration → POST /attendance/event {source: biometric, ...}
    │
    └── Face Scan (planned)
          └── Hardware integration → POST /attendance/event {source: face_scan, ...}

                        ↓ all sources write to attendance_events table

Processing Layer (resolves events → final record per staff per day)

                        ↓

{school_id}_staff_attendance (final daily record)

                        ↓

Admin Dashboard (exception-driven view)
    ├── Real-time snapshot: X present, Y absent, Z pending
    ├── Exception flags: who needs attention
    └── Correction actions: manual override for flagged records
```

---

## UX Screens Required

The UI must be redesigned around the exception-driven model, not the manual-marking model.

| Screen | Purpose |
|---|---|
| **Dashboard widget** | Real-time snapshot — X present, Y absent, Z pending action |
| **Attendance review screen** | Day view with exception flags; corrections are the primary action, not marking |
| **Settings** | Shift templates, geofence config, source management |

The current `staff_attendance_view.dart` is built manual-first (marking form). It needs to be redesigned as a review/exception screen. Manual marking becomes a secondary action (correction) accessed per staff record, not the primary screen action.

---

## Components

### 1. SMS-UI Flutter Shell (New Product)
Native Flutter app per school (white-labeled). Wraps sms-ui in a WebView.
- Handles GPS, background geofence, permissions, push notifications
- Communicates with WebView via JS bridge
- Built once, flavored per school (build-time white-labeling)
- **Status:** Needs to be built from scratch
- **Docs:** `/docs/sms-ui-shell/`

### 2. School Geofence Settings (sms-ui + sms-api)
Admin defines the school boundary (center coordinates + radius) in School Settings.
- Stored in school settings table
- Fetched by shell on login
- **Status:** Needs spec
- **Docs:** `docs/sms-ui/modules/needs-spec/geofence-school-settings.md`

### 3. Staff Shift Templates (sms-ui + sms-api)
Admin creates shift templates and assigns them to staff. Templates define:
- Shift start/end times and grace periods
- Working days
- Break windows (named time ranges with grace periods)
- Break windows are fetched by shell — exits during breaks are ignored
- **Status:** Needs spec
- **Docs:**
  - `docs/sms-ui/modules/needs-spec/staff-shift-templates.md`
  - `docs/modules/needs-spec/staff-shift-templates-api.md`

### 4. Attendance Rules Engine (sms-api)
Dynamic, school-configurable rules that evaluate attendance behavior and trigger automatic actions.
- Rules: late arrivals, early exits, break overruns, consecutive absences, etc.
- Actions: deduct leave, mark LOP, notify, flag for review, credit overtime
- Background job processes rules daily/monthly
- **Status:** Needs spec
- **Docs:**
  - `docs/sms-ui/modules/needs-spec/attendance-rules-engine.md`
  - `docs/modules/needs-spec/attendance-rules-engine-api.md`

### 5. Geofence Clock-in/out API (sms-api)
New endpoints to handle automated clock-in/out events from the Flutter shell.
- Extends existing `source` field: adds `geofence` as a source type
- Validates against shift template (break windows, expected times)
- **Status:** Needs spec
- **Docs:** `docs/modules/needs-spec/location-attendance-api.md`

---

## Existing Module (Built — Needs Redesign)

The current staff attendance module is built manual-first. Backend schema is largely reusable but the frontend UX needs to be redesigned to be exception-driven.

**Backend (reusable):**
- `check_in_time`, `check_out_time` — geofence/other sources populate these
- `source` field (`manual`, `machine`) — extend with `geofence`, `biometric`, `face_scan`
- Status enum (present/absent/late/half_day/holiday) — rules engine writes to this
- Full audit logging

**Frontend (needs redesign):**
- Current: manual marking form as primary action
- Required: exception review dashboard as primary action; manual correction as secondary

**Pending architectural decision:** whether to introduce a separate `attendance_events` table for raw source events, or continue using a single `staff_attendance` record with source field. This affects how multi-source conflicts are handled.

**Docs:** `docs/sms-ui/modules/built/attendance-module.md`

---

## Implementation Plan

Full step-by-step redesign instructions for DB schema, backend pipeline, and Flutter UX:

**`docs/product/planned/staff-attendance-redesign-plan.md`**

---

## Build Phases

### Phase 1 — Foundation
- SMS-UI Flutter Shell (WebView wrapper, white-label, auth bridge, JS bridge)
- School geofence settings UI + API

### Phase 2 — Shift Templates
- Staff shift template model + CRUD
- Break windows in templates
- Staff-to-template assignment

### Phase 3 — Automated Attendance
- Shell background geofence service
- Shell fetches coordinates + shift template on login
- Clock-in/out API endpoints (source: geofence)
- Break window awareness in shell

### Phase 4 — Rules Engine
- Rule builder UI
- Rule evaluation background job
- Action execution (leave deduction, LOP, notifications)
- Rule trigger audit log

---

## Key Decisions Made

| Decision | Choice | Reason |
|---|---|---|
| Mobile approach | Flutter WebView shell (not separate app) | sms-ui already web-responsive; staff uses web for complex tasks |
| Location decision | Flutter shell decides enter/exit | More reliable than browser geolocation API |
| White-labeling | Build-time per school (flavor) | Consistent with existing KYC shell approach |
| Background tracking | Shell handles, not browser | Browser cannot run background services reliably |
| Break window logic | Flutter shell ignores exits during breaks | Prevents false early clock-out triggers |
