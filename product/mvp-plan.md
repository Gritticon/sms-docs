---
title: MVP Plan — Gritticon SMS
type: product
project: platform
owner: PM
last-updated: 2026-06-06
---

# MVP Plan — Gritticon SMS

## What MVP Means Here

MVP = the platform is functional enough that a school can run their daily operations on it and we can sell it with confidence. The 9 modules below define the scope. Phase 2 (AI, KYC API wiring, Admissions automation) comes after.

---

## Gap Analysis — Where We Stand Today

Each module is scored against two axes: **API** (backend) and **UI** (Flutter web).

| # | Module | Sub-feature | API | UI | Gap Summary |
|---|--------|-------------|-----|----|-------------|
| 1 | Dashboard | Widgets + customisation | ✅ Built | ⚠️ Partial | View exists, individual widgets not implemented |
| 2 | Communication Hub | Announcements | ✅ Built | ⚠️ Partial | Placeholder view — needs real implementation |
| 2 | Communication Hub | Messages | ✅ Built | ⚠️ Partial | Conversation list + thread views exist, needs wiring |
| 2 | Communication Hub | Notifications | ✅ Built | ⚠️ Partial | Feature view exists, needs wiring |
| 2 | Communication Hub | Posts | ✅ Built | ⚠️ Partial | Create post view exists, needs completion |
| 3 | Student Management | Student Profiles | ✅ Built | ✅ Built | Complete — verify edge cases |
| 3 | Student Management | Promote Students | ✅ Built | ✅ Built | Model + controller + view exist — verify E2E |
| 4 | Staff Management | Staff Profiles | ✅ Built | ✅ Built | Complete |
| 4 | Staff Management | Attendance (manual) | ✅ Built | ⚠️ Redesign | Redesign in progress (geofence-first). **MVP scope: manual-only path must work cleanly.** |
| 4 | Staff Management | Payroll | ✅ Built | ⚠️ Redesign | 7+ views exist but redesign in progress — finalize workspace→draft→finalize flow |
| 4 | Staff Management | Leave Management | ✅ Built | ⚠️ Partial | API routes complete, overview view exists — needs E2E flow testing |
| 4 | Staff Management | Roles & Departments | ✅ Built | ✅ Built | Complete |
| 5 | Class Management | Classes & Sections | ✅ Built | ✅ Built | Complete |
| 5 | Class Management | Assignments | ⚠️ Partial | ❌ Missing | Homework exists in KYC diary API — no SMS-side assignment management UI |
| 5 | Class Management | Class Attendance | ✅ Built | ✅ Built | Complete — verify |
| 5 | Class Management | Subjects | ✅ Built | ✅ Built | Complete |
| 6 | Schedule Management | Timetable | ✅ Built | ✅ Built | Complete |
| 6 | Schedule Management | Teacher Allotment | ✅ Built | ✅ Built | Complete |
| 6 | Schedule Management | Holidays | ✅ Built | ✅ Built | Complete |
| 6 | Schedule Management | Substitute | ✅ Built | ✅ Built | Complete |
| 7 | School Management | School Info | ✅ Built | ✅ Built | View exists — verify wiring |
| 7 | School Management | School Settings | ✅ Built | ✅ Built | View exists — verify wiring |
| 7 | School Management | KYC Settings | ✅ Built | ✅ Built | View exists but undocumented — spec + verify |
| 8 | **Admissions** | Leads | ❌ None | ❌ None | **Completely greenfield — no API, no UI, no spec** |
| 8 | **Admissions** | Enquiries | ❌ None | ❌ None | Greenfield |
| 8 | **Admissions** | In Progress | ❌ None | ❌ None | Greenfield |
| 9 | Fee Management | Fee Templates | ❌ None | ❌ None | **Not built** — student_fees.py is per-student records only, no templates |
| 9 | Fee Management | Concessions | ❌ None | ❌ None | Not built |
| 9 | Fee Management | Fee Collections | ⚠️ Partial | ❌ None | Per-student fee CRUD exists, no management UI |

### Summary

| Status | Modules |
|--------|---------|
| ✅ Complete (verify only) | Schedule Management (all 4), Student Profiles, Student Promotion, Staff Profiles, Roles & Departments, Classes, Subjects, Class Attendance, School Management |
| ⚠️ Partial (needs completion) | Dashboard, Communication Hub, Staff Attendance, Payroll, Leave Management, Assignments |
| ❌ Greenfield (build from scratch) | **Admissions** (entire module), **Fee Templates**, **Concessions** |

---

## Sprint Plan

**Assumption:** 1 developer on Flutter + 1 developer on Python API. Adjust sprint capacity if more/fewer.
**Sprint length:** 2 weeks each.
**Total timeline to MVP:** ~20–22 weeks (~5 months).

---

### Sprint 1 — Foundation & Verification (Weeks 1–2)

**Goal:** Confirm everything marked "complete" actually works E2E before building on top of it.

**Work:**
- Smoke test all "complete" modules end-to-end: Classes, Subjects, Timetable, Teacher Allotment, Holidays, Substitute, Staff Profiles, Roles & Departments
- School Management: verify school_info, school_settings, kyc_settings views are fully wired to API — fix any broken connections
- Student Profiles: verify all tabs (Academic Records, Achievements, Certificates, CoC, Documents) load correctly
- Student Promotion: run E2E — promote a class, verify data changes correctly

**Done when:**
- All verified modules pass a manual test checklist with no broken flows
- School Management is fully functional (info editable, settings saved, KYC settings toggle correctly)

---

### Sprint 2 — Staff Management: Leave + Attendance (Weeks 3–4)

**Goal:** Complete the two in-progress staff management features.

**Work:**
- **Leave Management E2E:** Setup leave types → Staff applies for leave → Admin views in leave_overview → Admin approves/rejects → Attendance record updates (on_leave status) → Leave balance reflects correctly. Fix any broken links in this chain.
- **Staff Attendance (manual path):** Strip away all geofence dependencies for MVP. The manual mark/edit path must work cleanly and reliably. Exception review UI should be functional even without geofence data feeding it.
- Document both flows once verified.

**Done when:**
- A staff member can apply for leave and an admin can approve it — attendance and balance both update
- An admin can manually mark/edit staff attendance for any day without errors

---

### Sprint 3 — Staff Management: Payroll (Weeks 5–6)

**Goal:** Complete the payroll redesign and make it production-ready.

**Work:**
- Finalize the workspace → draft → approve → finalize payroll flow
- Salary templates: verify they apply correctly to staff
- Advances: verify deduction from payslip
- LOP (Leave Without Pay): verify it feeds from approved leaves
- Payslip: staff can view their own payslip in their profile
- Fix any broken connections between payroll and leave data

**Done when:**
- Admin can run a complete monthly payroll cycle from start to finalized payslip
- Staff can view their payslip

---

### Sprint 4 — Class Management: Assignments (Weeks 7–8)

**Goal:** Build the missing Assignments feature in SMS-UI.

**Spec to define before build:**
- Teacher creates an assignment: title, description, subject, class/section, due date, optional attachment
- Assignments list view per class: filterable by subject, status, date
- Edit/delete assignment
- On the API side: assignments table (school-scoped), CRUD routes, permission gating
- KYC diary already displays homework — the SMS assignments API is the source for that

**Work:**
- API: design assignments table + CRUD endpoints
- UI: assignments list view + create/edit flow
- Wire to KYC diary (assignments created in SMS appear as homework in KYC)

**Done when:**
- Teacher can create an assignment in SMS
- It appears in KYC diary for the relevant students

---

### Sprint 5 — Communication Hub (Weeks 9–10)

**Goal:** Complete all four Communication Hub features.

**Work:**
- **Announcements:** Replace placeholder with real list view — create, view, target (all staff / specific role / class), delete
- **Messages:** Wire conversation list + thread view to API — send, receive, mark read
- **Notifications:** Wire notifications view to API — list, mark read, filter
- **Posts:** Complete create post flow — title, body, media upload, category, publish. List and detail views.

**Done when:**
- Admin can post an announcement — staff see it
- Staff can message each other
- Notifications populate from system events
- Admin can create a post — it appears in KYC home feed

---

### Sprint 6 — Fee Management (Weeks 11–13) *[3 weeks]*

**Goal:** Build Fee Management from scratch — the biggest missing module and a deal-breaker for Indian schools.

**Spec (define Week 11, build Weeks 12–13):**

**Fee Templates:**
- Template = a named fee structure (e.g. "Term 1 2026") containing line items (Tuition, Transport, Lab, etc.) with amounts
- Templates are created once and assigned to student groups (class/section or individual)
- Support for instalment schedules (due dates per term)

**Concessions:**
- Named concession types (Staff Ward, Merit, SC/ST, Sibling Discount, etc.)
- Flat amount or percentage discount
- Applied to a specific student's fee assignment
- Audit trail: who applied the concession, when, why

**Fee Collections:**
- Per-student fee view: dues, paid, outstanding, concessions applied
- Record a payment: amount, date, mode (cash/UPI/bank transfer), reference
- Receipt generation (PDF)
- Bulk view: which students have outstanding fees across a class/section

**API:** New tables — fee_templates, fee_line_items, fee_assignments, fee_concessions, fee_payments
**UI:** Fee Templates screen, Concessions setup, Collections workspace

**Done when:**
- Admin can create a fee template, assign it to a class, apply a concession to a student, and record a payment
- Outstanding fees are visible per student and per class

---

### Sprint 7 — Admissions Module (Weeks 14–16) *[3 weeks]*

**Goal:** Build the Admissions pipeline from scratch — completely new module.

**Spec (define Week 14, build Weeks 15–16):**

**Three stages (pipeline model):**

1. **Leads** — Initial contact/interest. Fields: name, contact number, enquiry date, child's name, grade applying for, source (walk-in, phone, referral, website), notes. No formal application yet.

2. **Enquiries** — School has engaged. Additional fields: parent name, email, current school, preferred session, documents submitted (checklist). Status: Pending → Scheduled (for assessment/interview) → Completed.

3. **In Progress** — Formal application underway. Fields: application form, assessment result, fee payment status (registration fee), offer letter sent, acceptance confirmed. Status moves through: Applied → Assessed → Offered → Accepted → Enrolled.

**Conversion flow:** Lead → Enquiry → In Progress → Enrolled (creates a Student Profile on enrolment).

**API:** New tables — admissions_leads, admissions_enquiries, admissions_applications (all school-scoped)
**UI:** Kanban or tabbed pipeline view, detail views per stage, stage-transition actions

**Done when:**
- Admin can track a prospect from first enquiry all the way to enrolment
- On enrolment, a student profile is created with pre-filled data from the admission record

---

### Sprint 8 — Dashboard (Weeks 17–18)

**Goal:** Build the Flutter dashboard widget system. This is last because it depends on all modules working correctly (it pulls live data from them).

**Work:**
- Implement all widgets from the spec in `docs/sms-ui/modules/built/dashboard-module.md`
- Personal widgets first (my timetable, my attendance, my classes today, my subjects)
- Administrative widgets per module (student count, staff attendance, upcoming exams, etc.)
- Widget picker + customization flow (select-and-place reorder, add/remove)
- Save/restore layout via `GET/PUT /staff/me/dashboard`
- Add `fl_chart` dependency for chart widgets

**Done when:**
- Any staff member sees a relevant dashboard on login with correct live data
- They can customise and save their layout
- Permission-gated widgets show/hide correctly by role

---

### Sprint 9 — QA, Polish & Pilot Prep (Weeks 19–20)

**Goal:** Harden the product for the first paying school.

**Work:**
- Full E2E regression pass across all 9 modules
- Permission matrix testing: every role sees only what they should
- Multi-tenant check: school data does not bleed across schools
- Performance baseline: list screens load in < 2 seconds
- Empty state handling: every screen has a valid empty state
- Error handling: network errors, validation errors surface clearly
- Fix critical bugs from QA
- Prepare onboarding checklist for first school (what Gritticon admin panel needs to configure)

**Done when:**
- All 9 modules pass E2E test with no P1/P2 bugs open
- A test school has been fully set up and used for 1 week internally

---

## Sprint Summary

| Sprint | Focus | Weeks | Size |
|--------|-------|-------|------|
| 1 | Verification + School Management | 1–2 | Small |
| 2 | Leave Management + Staff Attendance | 3–4 | Medium |
| 3 | Payroll | 5–6 | Medium |
| 4 | Assignments | 7–8 | Medium |
| 5 | Communication Hub | 9–10 | Medium |
| 6 | Fee Management | 11–13 | Large |
| 7 | Admissions | 14–16 | Large |
| 8 | Dashboard | 17–18 | Medium |
| 9 | QA + Pilot Prep | 19–20 | Medium |

**Total: 20 weeks (~5 months)**

---

## Risk Register

| Risk | Severity | Mitigation |
|------|----------|------------|
| Admissions is completely greenfield | High | Spec in Week 14 before any code. Avoid scope creep — keep it simple pipeline, no AI, no automation. |
| Fee Management has no existing foundation | High | Start spec in Week 11. Don't boil the ocean — MVP is manual collections only, no payment gateway. |
| Payroll redesign has multiple in-flight changes | Medium | Freeze scope in Sprint 3. Document exactly what "done" looks like before coding starts. |
| Staff Attendance geofence redesign creates regression risk on manual path | Medium | Explicitly isolate and test manual path in Sprint 2. Keep geofence as a future enhancement. |
| KYC API has zero wiring (UI exists, no backend calls) | Medium | Not an MVP blocker for SMS. Flag as Sprint 10 post-MVP — but assignments (Sprint 4) must wire correctly as it feeds KYC diary. |
| No payment gateway for fee collection | Low | MVP = manual payment recording + receipts. Online payment is Phase 2. State this clearly to pilot schools. |
| Dashboard dependencies on all modules | Low | Building it last (Sprint 8) mitigates this. Each widget degrades gracefully with empty state. |

---

## Definition of Done (Module Level)

A module is MVP-ready when:

1. All specified features work E2E without errors
2. Permissions gate access correctly — role without access cannot see or action anything
3. Every list screen has a working empty state
4. Every form validates inputs before submission
5. Create / edit / delete operations have confirmation where destructive
6. Audit logs record who did what (for modules that modify records)
7. API returns correct error messages for bad input
8. Screen loads in < 2 seconds on a standard connection

---

## How I'll Operate as PM

**Weekly cadence:**
- Monday: Sprint planning / backlog review — what's being built this week, in what order
- Wednesday: Mid-week check — any blockers? Scope questions resolved same day
- Friday: Demo of what's been built — working feature only, no WIP

**My responsibilities:**
- Spec every new feature before a developer touches it (I write the spec, developer implements)
- Maintain this plan — update sprint status weekly, flag slippage early
- Make scope decisions fast — if something is scope creep, it gets cut or deferred
- Own the Definition of Done — I do the acceptance test, not just the developer

**Developer responsibilities:**
- Build against the spec, not assumptions
- Flag blockers same day — don't sit on a problem for 2 days
- No feature is "done" until PM acceptance test passes

**What I won't do:**
- Add scope mid-sprint without removing something else
- Accept "it works on my machine" as done
- Let a spec ambiguity block development for more than 24 hours

---

## First Decision: Admissions Priority

One call to make before Sprint 1 starts:

**Should Admissions (Sprint 7) move earlier in the roadmap?**

The case for moving it up: Admissions is the first workflow a school runs before any other module matters. A school evaluating the product will likely ask "can I manage my admissions?" first.

The case for keeping it late: It's completely greenfield and at risk of scope creep. Building it after the core operational modules are stable reduces integration risk.

**My recommendation:** Keep it at Sprint 7 unless you're targeting a school that is actively in admissions season when you pilot. If that's the case, move it to Sprint 4–5 and push Communication Hub and Dashboard to Sprint 8–9.
