---
title: Module Planning Guide — Lessons from Attendance, Leave & Payroll
type: reference
project: platform
last-updated: 2026-03-29
---

# Module Planning Guide
### What went wrong in planning, why, and how to prevent it next time

This document captures every correction made during the review of the Attendance, Leave, and Payroll redesign plans. Each lesson is derived from a real gap that was only caught in review — not during initial planning.

Use this as a checklist before finalising any future module plan.

---

## Part 1 — Changes Made and Why

### Attendance Plan

| Change | Why |
|---|---|
| Renamed "Attendance Setup" → "Setup" with extensible Sources structure | Named after current content (geofence only) — would have needed renaming the moment biometric was added. Name should reflect function, not content. |
| Added Assigned Staff tab to Shift Templates screen | Shift assignment was only accessible from one direction (template → setup). Admin creating a new staff member had no way to assign a shift without leaving Staff Management and navigating to Setup. |
| Added `effective_from` date picker and auto-close logic to shift assignments | Without this, changing a staff member's shift would create a conflict in the `staff_shift_assignments` table — old and new assignments both active with no end date on the old one. |
| Added shift template fallback to processing function | If a staff member has no shift assignment, the processing function had no defined behaviour. Would throw an error or silently skip shift-based logic. Default template fallback and exception flagging were both missing. |
| Noted `on_leave` and `leave` source enum extensions as owned by leave plan | Leave plan adds new values to attendance enums — this cross-plan dependency was invisible until both plans were read together. |

---

### Leave Plan

| Change | Why |
|---|---|
| Step 3.3: Approvals moved to "My Profile → Approvals tab" | Original location was "notifications or a dedicated Approvals tab" — ambiguous, uncommitted. Without a defined location, the screen has no navigation home and cannot be deep-linked from notifications. |
| Added Step 3.3b: Admin Staff Approvals audit tab | No plan for admin to audit who approved what on a per-staff basis. This is an HR oversight need that was completely missing. |
| Step 3.1: "staff's personal section" → "My Profile → Leaves & Permissions tab" | "Personal section" was used across three screens in two plans without ever being defined. Ambiguous location = no navigation home = inconsistent implementation. |
| Step 3.2: My Early Permissions merged into same tab as My Leaves | Leave and early permissions are the same mental model from the staff's perspective ("my requests"). Splitting them into separate screens adds friction with no benefit. |
| Step 3.4: Leave Overview explicit tab structure (Overview + Requests) | The plan implied a tabbed structure but never stated it. "Requests tab" was referenced without a parent container being defined. |
| Step 3.6: Leave config location updated to "Setup → Leave" | Referenced "Attendance Setup section" — which no longer exists after the rename. Config location was stale the moment the Setup section was restructured. |
| Added push notification requirement to Step 3.3 | The entire approval flow breaks without notifications — approvers have no reason to open their profile tab proactively. Notification is the trigger; the tab is the destination. Designed the screen without designing the entry point. |
| Removed `early_permission_handling` from `leave_types` table | Same field existed in both `leave_types` (Step 1.2) and `early_permission_config` (Step 1.6). Two sources of truth for the same config = future inconsistency when they diverge. |

---

### Payroll Plan

| Change | Why |
|---|---|
| Added Step 5.1b: Salary Templates management screen with Assigned Staff tab | Salary templates existed as a concept but had no management screen. Admin had no way to see which staff are on which template or reassign in bulk — only individual changes via the payslip drawer. |
| Added "flag affected payslips on reassignment" rule | Bulk reassigning staff to a new template during an open payroll run would silently recalculate payslips — overwriting amendments admin had already made. Needs a confirmation step, not silent action. |
| Step 5.6: "personal section" → "My Profile → Payslips tab" | Same "personal section" ambiguity as leave plan. |

---

### Cross-Plan Gaps

| Gap | Why it was missed | Fix applied |
|---|---|---|
| `is_lop_flagged` missing from `staff_attendance` | Leave plan defined it on `leave_requests`. Payroll plan assumed it was on `staff_attendance`. Each plan was written in isolation — the data contract between them was never mapped. | Add field to `staff_attendance`; leave pipeline writes it when creating attendance record. |
| Early permission monthly summary table never defined | Leave plan said "write to a monthly summary". Payroll plan said "read the monthly summary". Neither plan defined the table. Referenced without being designed. | Define table schema in leave plan between Steps 2.6 and Part 3. |
| Cross-plan prerequisites not stated | Each plan has its own implementation order. No plan states "this requires X plan to be built first". A developer starting with the payroll plan would hit missing tables and enums. | Add prerequisites section to leave and payroll plan headers. |
| Cancelled approved leave attendance revert undefined | Only pending cancellation was described. Approved leave that admin reverses leaves stale `on_leave` records in attendance with no resolution path. | Note in leave Step 2.3: same manual correction rule as rejected leave applies. |

---

## Part 2 — Planning Lessons

These are the root causes. Each one maps to multiple changes above.

---

### Lesson 1 — Every screen needs a precise navigation address before you describe its contents

**What went wrong:** "Accessible from the staff's personal section" appeared three times across two plans. "Notifications or a dedicated Approvals tab" was written for the approvals screen. Neither was a real navigation address.

**The rule:** Before writing a single bullet point about what a screen contains, write its full location path:
```
Location: My Profile → Leaves & Permissions tab
```
If you cannot write this path, the screen does not have a home yet. Stop and decide where it lives first.

**Also plan:** Every screen that requires user-initiated action needs its entry point defined — the notification, badge, or deep link that brings the user there. A screen without an entry point will not be used.

---

### Lesson 2 — Cross-module data contracts must be explicitly mapped

**What went wrong:** The leave plan wrote `is_lop_flagged` on `leave_requests`. The payroll plan read `is_lop_flagged` from `staff_attendance`. Each plan was internally consistent but they disagreed on where the field lives. The early permission monthly summary was referenced by two plans but defined by neither.

**The rule:** When Module A produces data that Module B consumes, the producing module's plan must define:
- The exact table and field name
- Who writes it and when
- Who reads it and what query they use

Draw the contract before writing either module's plan. A pipeline is only as strong as its joints.

---

### Lesson 3 — Configuration sections must be named for function, not current content

**What went wrong:** "Attendance Setup" was named after what was in it at the time (attendance config). The moment leave configuration was added, the name was wrong. If biometric configuration is added next, it would be wrong again.

**The rule:** Name configuration sections by their role in the system:
- "Setup" — for school-level configuration of how the system operates
- Not "Attendance Setup", not "Leave Setup", not "HR Config"

Structure the section so new modules plug in without renaming. The shape of the section should mirror the pipeline — if a new source is added to the `attendance_events.source` enum, there is already a place for its config to live.

---

### Lesson 4 — Every many-to-many relationship needs UX in both directions

**What went wrong:** Shift templates could only be assigned to staff from one direction: Setup → Shift Templates → Assigned Staff. Admin creating a new staff member had no way to assign a shift without leaving their current context. Same gap existed for salary templates.

**The rule:** For any assignment relationship (template ↔ staff, role ↔ permission, shift ↔ department), plan both directions:
- From the template side: who is assigned to this? (bulk management)
- From the staff side: what is this person assigned to? (individual management)

Both directions will be used. Plan both.

---

### Lesson 5 — State changes in one module that affect another module must be explicitly traced

**What went wrong:** Approved leave creates attendance records. Cancelled or reversed approved leave leaves those attendance records stale. The plan described leave state changes without tracing their downstream impact on attendance.

**The rule:** For every status transition in a module (pending → approved → cancelled), ask:
- Does this touch another module's data?
- What is the downstream module's state after this transition?
- Is a revert automatic or manual?

Document the answer even if the answer is "admin corrects manually." Silence is not documentation.

---

### Lesson 6 — Single source of truth — if two places can answer the same question, remove one

**What went wrong:** `early_permission_handling` appeared on both `leave_types` (as a column) and `early_permission_config` (as a per-leave-type override row). Both could answer "how should early permission be handled for this leave type?" When they disagreed, which would win?

**The rule:** Before adding a config field, ask: does this already exist somewhere else in the schema? If yes — route everything through the existing field or delete the duplicate. Two sources for the same config always diverge eventually.

---

### Lesson 7 — Read all related plans together before finalising any one plan

**What went wrong:** Each plan was written and reviewed in isolation. The cross-plan gaps (`is_lop_flagged`, summary table, enum ownership) were only visible when all three plans were read simultaneously.

**The rule:** Before marking any plan as ready for implementation:
1. Read it alongside every plan it depends on and every plan that depends on it
2. Walk the full pipeline from raw input to final output across all modules
3. Verify every table/field referenced by Plan B actually exists in Plan A's schema

A plan that looks complete in isolation may have broken joints when connected.

---

### Lesson 8 — Implementation orders must state cross-plan prerequisites

**What went wrong:** Each plan had its own numbered implementation order. None stated "this plan requires X plan to be built first." A developer could start payroll before attendance exists.

**The rule:** Every plan that depends on another plan must start with:
```
## Prerequisites
This plan requires the following plans to be fully implemented first:
- [Plan name] — [specific tables/functions needed]
```

---

### Lesson 9 — Know the difference between configuration that grows and configuration that is fixed

**What went wrong:** Attendance sources (geofence, biometric, face scan) are known to grow. The setup structure should have been designed to accommodate this from day one — a "Sources" sub-section where each source plugs in independently. Instead it was designed as a flat list that would need restructuring for each new source.

**The rule:** When planning a configuration section, identify:
- Will new items of this type be added in future? (Yes → design as extensible list)
- Is this a fixed set of options? (Yes → design as a settings form)

Geofence is one of several attendance sources → Sources list pattern.
Working days per month is a fixed setting → settings form pattern.

---

## Part 3 — Pre-Finalisation Checklist for Any Module Plan

Run through this before marking a plan ready for implementation.

### Navigation
- [ ] Every screen has a full location path (e.g., `My Profile → Leaves & Permissions tab`)
- [ ] Every screen requiring user action has its entry point defined (notification, badge, deep link)
- [ ] Configuration sections are named for function, not current content
- [ ] New module has a slot in the Setup section without requiring renaming

### Data Contracts
- [ ] Every field produced by this module and consumed by another module is explicitly named and located
- [ ] Every field consumed by this module and produced by another module is verified to exist in that other plan
- [ ] No "write to a summary that X reads" without defining the summary table

### State & Transitions
- [ ] Every status transition traced for downstream module impact
- [ ] Revert/cancellation behaviour documented (automatic or manual) for every approval flow
- [ ] "Admin corrects manually" is an acceptable answer — but must be written, not assumed

### Schema
- [ ] No duplicate config fields across tables answering the same question
- [ ] All enum extensions owned and noted in both the defining plan and the referencing plan
- [ ] Assignment relationships have both directions of UX planned

### Integration
- [ ] Prerequisites section written at the top of the plan
- [ ] Plan reviewed alongside all plans it depends on and all plans that depend on it
- [ ] Full pipeline walked: raw input → event → processing → final record → downstream consumer
