---
title: Cross-Project Data Flow
type: architecture
project: platform
last-updated: 2026-03-18
---

# Cross-Project Data Flow

This document maps how data moves between `sms-ui`, `sms-api`, and `kyc` for every module. It answers the question: *when staff do X in sms-ui, what do parents/students see in kyc — and vice versa?*

All data flows through `sms-api`. There is no direct communication between `sms-ui` and `kyc`.

---

## Flow Direction Key

| Symbol | Meaning |
|---|---|
| `sms-ui → kyc` | Staff creates/manages data in sms-ui; parent/student views it in kyc |
| `kyc → sms-ui` | Parent/student creates data in kyc; staff views/manages it in sms-ui |
| `↔ bidirectional` | Both sides create and read data for the same feature |

---

## Module Data Flows

### Student Management

| Submodule | sms-ui Action | kyc View/Action | Direction |
|---|---|---|---|
| Student Profiles | Admin creates and manages student profiles | Student/Parent views student profile | `sms-ui → kyc` |
| Attendance | Teacher/Admin marks student attendance | Parent views attendance history and summaries | `sms-ui → kyc` |
| Achievements | Admin records student achievements | Parent/Student views achievements | `sms-ui → kyc` |
| Certificates | Admin manages student certificates | Parent/Student views and downloads certificates | `sms-ui → kyc` |
| Documents | Admin uploads student documents | Parent/Student views and downloads documents | `sms-ui → kyc` |
| Code of Conduct | Admin records code of conduct entries | Parent/Student views code of conduct record | `sms-ui → kyc` |
| House Tag | Admin assigns house tags to students | Parent/Student views house tag | `sms-ui → kyc` |
| Requests | Admin views and responds to requests | Parent raises requests | `↔ bidirectional` |

**Requests detail:** Parent submits a request via kyc → appears in sms-ui Requests submodule (311) for staff to review and respond → response visible back to parent in kyc.

---

### Exam Management

| Submodule | sms-ui Action | kyc View/Action | Direction |
|---|---|---|---|
| Exams | Admin creates exam schedules | Student/Parent views upcoming exams | `sms-ui → kyc` |
| Results | Admin enters exam results | Student/Parent views results and marks | `sms-ui → kyc` |
| Report Cards | Admin generates report cards | Parent/Student views and downloads report cards | `sms-ui → kyc` |
| Question Papers | Admin manages question papers | Not exposed in kyc | `sms-ui only` |

---

### Class Management

| Submodule | sms-ui Action | kyc View/Action | Direction |
|---|---|---|---|
| Classes & Sections | Admin manages class structure | Not directly exposed (inferred from student profile) | `sms-ui only` |
| Subjects | Admin assigns subjects to classes | Student views their subjects | `sms-ui → kyc` |
| Assignments | Teacher creates assignments for a class | Student views assignments for their class | `sms-ui → kyc` |
| Class Attendance | Teacher marks class-level attendance | Parent views class attendance | `sms-ui → kyc` |

---

### Schedule Management

| Submodule | sms-ui Action | kyc View/Action | Direction |
|---|---|---|---|
| Timetable | Admin generates and publishes school timetable | Student views their own timetable; Teacher views their own timetable (personal widget in sms-ui) | `sms-ui → kyc` |
| Holidays | Admin creates school holidays | Parent/Student views upcoming holidays | `sms-ui → kyc` |
| Teacher Allotment | Admin assigns teachers to classes | Not exposed in kyc | `sms-ui only` |
| Substitute | Admin manages substitute teachers | Not exposed in kyc | `sms-ui only` |

---

### Communication Hub

| Submodule | sms-ui Action | kyc View/Action | Direction |
|---|---|---|---|
| Announcements | Admin creates school-wide announcements | Parent/Student views announcements | `sms-ui → kyc` |
| Messages | Staff send and receive messages | Parent/Student sends and receives messages | `↔ bidirectional` |
| Notifications | System-generated based on events | Parent/Student views notifications | `sms-ui → kyc` |

**Messages detail:** Fully bidirectional — staff can initiate conversations with parents, parents can initiate with staff. Both sides send and receive through the same messaging system via sms-api.

---

### Complaints

| Flow | sms-ui Action | kyc Action | Direction |
|---|---|---|---|
| Parent complaint | Staff views and manages complaint | Parent raises complaint about a situation | `kyc → sms-ui` |
| Teacher complaint | Teacher raises complaint on a student | Parent views complaint raised about their child | `sms-ui → kyc` |
| Resolution | Staff resolves the complaint | Parent views resolution | `sms-ui → kyc` |

**Complaints detail:** Fully bidirectional. Teachers can raise complaints on students via sms-ui. Parents can raise complaints via kyc. All complaints are managed by staff in sms-ui (Complaints module, submodules 701 and 702).

---

### Transport Management

| Submodule | sms-ui Action | kyc View/Action | Direction |
|---|---|---|---|
| Routes | Admin manages bus routes | Parent views assigned route | `sms-ui → kyc` |
| Vehicles | Admin manages vehicles | Parent views assigned vehicle | `sms-ui → kyc` |
| Drivers | Admin manages drivers | Parent views assigned driver | `sms-ui → kyc` |
| Bus Tracking | — | Parent tracks bus location live (planned) | `sms-api → kyc` |

---

### Class Session

| Submodule | sms-ui Action | kyc View/Action | Direction |
|---|---|---|---|
| Sessions | Teacher manages class sessions | Student views session details and schedule | `sms-ui → kyc` |
| Class Progress | Teacher records class progress | Student/Parent views class progress | `sms-ui → kyc` |

---

### Staff Management

| Submodule | sms-ui Action | kyc View/Action | Direction |
|---|---|---|---|
| Staff Profiles | Admin manages staff records | Not exposed in kyc | `sms-ui only` |
| Staff Attendance | Admin/Staff marks staff attendance | Not exposed in kyc | `sms-ui only` |
| Payroll | Admin manages payroll | Not exposed in kyc | `sms-ui only` |
| Roles & Departments | Admin manages roles | Not exposed in kyc | `sms-ui only` |

---

### School Management

| Submodule | sms-ui Action | kyc View/Action | Direction |
|---|---|---|---|
| School Info | Admin manages school details | School name and basic info shown in kyc profile | `sms-ui → kyc` |
| School Settings | Admin manages settings | Not exposed in kyc | `sms-ui only` |

---

### Dashboard (sms-ui only)

The dashboard is exclusive to sms-ui. It is not exposed in kyc.

---

## Features Exclusive to kyc (no sms-ui counterpart)

These are features that originate in kyc without a direct management counterpart in sms-ui.

| Feature | Note |
|---|---|
| Diary | Parents/students write diary entries. Currently no sms-ui management view. |
| Fees & Payments (planned) | Parents make payments via kyc. Finance management may require a future sms-ui module. |
| Online Orders (planned) | Parents place orders via kyc. Order management may require a future sms-ui module. |
| Library (planned) | Students browse and request books via kyc. Library management TBD for sms-ui. |
| Reminders (planned) | System-generated reminders in kyc. Managed via notifications in sms-ui. |

> **Note for sms-ui planning:** As kyc planned features become built, evaluate whether a corresponding management module is needed in sms-ui. Document this in `product/feature-matrix.md`.

---

## Impact Rules for Data Flow

When making changes, use this checklist:

**If changing a sms-ui feature that writes data:**
- [ ] Does this data appear in kyc? (check the table above)
- [ ] If yes, does the kyc view still work correctly after the change?
- [ ] Update both the sms-ui module doc and the kyc spec doc

**If adding a new sms-ui feature that writes data:**
- [ ] Should this data be visible in kyc?
- [ ] If yes, create a kyc spec in `kyc/specs/planned/` and register it
- [ ] Update the data flow table in this document

**If adding a new kyc feature that creates data:**
- [ ] Does staff need to see or manage this data in sms-ui?
- [ ] If yes, create a sms-ui module doc in `sms-ui/modules/planned/` and register it
- [ ] Update the data flow table in this document

**If removing a feature from sms-ui:**
- [ ] Does removing it break the kyc counterpart view? (data will no longer exist)
- [ ] Update or archive the kyc spec if the data source is gone

---

## Related Documents

- `architecture/cross-project-dependencies.md` — dependency map and impact rules
- `product/feature-matrix.md` — which features exist in which projects
- `sms-ui/modules/built/` — staff-side module docs
- `kyc/specs/built/` — parent/student-side spec docs
