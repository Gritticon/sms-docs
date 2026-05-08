---
title: How to Plan the Attendance Feature
type: spec
project: kyc
module: Attendance
last-updated: 2026-04-08
---


> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

# How to Plan the Attendance Feature — Product/Engineering Note

## 1. Summary

Plan Attendance using the same **lifecycle and doc patterns** as Diary and Timetable: **UI-first in planned**, implement with stubs, integrate SMS APIs, validate, then **snapshot to built**. The **built** spec (`kyc/docs/built/attendance-analytics-spec.md`) is the frozen release snapshot and defines scope (F1–F8, NF1–NF4), user stories, dependencies, risks, edge cases, and MVP (monthly/period views, summary metrics, basic low-attendance alerts; out of scope: complex analytics, editing/justification). Use it as the single source of truth for *what* to build; add **planned** artifacts to define *how* to build it (screens, components, build order, acceptance checklist) and to support future edits in planned before the next snapshot to built.

---

## 2. Planning Artifacts to Add

| Location | Document | Purpose |
|----------|----------|--------|
| **planned** | `attendance-ui-build-outline.md` | Confirmed assumptions, entry & flow summary, **screens & components in UI build order**, states (loading, empty, error), copy/labels, high-level backend dependencies. Mirrors structure of `diary-ui-build-outline.md` and `timetable-ui-build-outline.md`. |
| **planned** | `attendance-ui-pm-addendum.md` | Traceable **acceptance criteria checklist** (one AC per testable behavior) and **dependencies** (SMS attendance API, student/active-child context). Source: build outline. |
| **planned** (optional) | `attendance-analytics-spec.md` | Copy of the built spec for **editing future scope**; when a release is done, snapshot back to built. If the team prefers a single source of truth in built, skip this and edit built only at release boundaries. |

**Built** already has `attendance-analytics-spec.md`; do **not** duplicate the build outline or addendum in built until the feature is implemented and snapshotted (per lifecycle).

---

## 3. Recommended Phasing / UI Build Order

- **Phase 1 — Period selector & list/calendar**
  - Month/period selector (prev/next, date picker or period drop-down) and default to current month.
  - Calendar or list view showing each day’s status (present/absent/other) for the selected period.
  - Loading, empty (“No attendance records yet”), and error + retry.
- **Phase 2 — Summary metrics**
  - Summary block: total days present, total days absent, overall attendance % for the selected period (F4).
  - Ensure it updates when period or active child changes.
- **Phase 3 — Day drill-down**
  - Tap a day → detail screen: subject/period-wise status and remarks if API provides; otherwise overall status + notes (F5, Story 3).
- **Phase 4 — Alerts**
  - Surface low-attendance (or backend risk) flags as clear indicators or messages when API provides them (F7).

**API dependency:** Implementation can start with **stubs/mock data** (e.g. fixture month + per-day status). Full behavior depends on **SMS API attendance endpoints** (daily + aggregated per student, optional subject/period, risk flags) and **student profile / active-child** context; integrate when API contracts are ready and then validate against the build outline and PM addendum before snapshot to built.

---

## 4. Ties to the Lifecycle

- **Draft/design:** Build outline + addendum live in **planned**; they reference the built spec for requirements and MVP scope.
- **Implement UI:** Flutter builds in the order above, with stubs where APIs are not ready; loading visible for every request.
- **Integrate APIs:** Swap stubs for real SMS attendance + student context; update planned docs if UX or scope changes.
- **Validate & release:** QA checks against build outline + addendum (and built spec); when released, **snapshot** the spec (and optionally the outline/addendum) to **built** with date/version.
- **Evolve:** For later changes, edit the **planned** versions; re-snapshot to built after the next release.
