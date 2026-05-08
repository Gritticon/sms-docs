---
title: Feature Matrix
type: product
project: platform
last-updated: 2026-03-29
---

# Feature Matrix

This document maps every platform feature across all four projects. It answers at a glance: *what exists where, and what is still planned?*

Use this alongside `architecture/data-flow.md` to understand not just where a feature exists but how data moves between the projects that implement it.

---

## Status Key

| Symbol | Meaning |
|---|---|
| ✅ Built | Feature is implemented and live |
| ✅ 🔄 Redesign | Feature exists but undergoing active redesign (Sprints 1–8) |
| 🔲 Planned | Feature is designed but not yet built |
| — | Not applicable or not exposed in this project |
| 📋 Content | Website describes this feature in marketing copy |

---

## Feature Matrix

| Feature | sms-api | sms-ui | kyc | website |
|---|---|---|---|---|
| **STUDENT MANAGEMENT** | | | | |
| Student Profiles | ✅ | ✅ Admin manages | ✅ Parent/Student views | 📋 |
| Student Attendance | ✅ | ✅ Staff marks | ✅ Parent views | 📋 |
| Achievements | ✅ | ✅ Admin records | ✅ Parent/Student views | 📋 |
| Certificates | ✅ | ✅ Admin manages | ✅ Parent/Student views | 📋 |
| Documents | ✅ | ✅ Admin uploads | ✅ Parent/Student views | 📋 |
| Code of Conduct | ✅ | ✅ Admin records | ✅ Parent/Student views | 📋 |
| House Tag | ✅ | ✅ Admin assigns | ✅ Parent/Student views | — |
| Requests | ✅ | ✅ Staff manages | ✅ Parent raises | — |
| **EXAM MANAGEMENT** | | | | |
| Exam Schedules | ✅ | ✅ Admin creates | ✅ Student/Parent views | 📋 |
| Results / Marks | ✅ | ✅ Admin enters | ✅ Student/Parent views | 📋 |
| Report Cards | ✅ | ✅ Admin generates | ✅ Parent/Student views | 📋 |
| Question Papers | ✅ | ✅ Admin manages | — | — |
| **CLASS MANAGEMENT** | | | | |
| Classes & Sections | ✅ | ✅ Admin manages | — | — |
| Subjects | ✅ | ✅ Admin assigns | ✅ Student views | 📋 |
| Assignments | ✅ | ✅ Teacher creates | ✅ Student views | 📋 |
| Class Attendance | ✅ | ✅ Teacher marks | ✅ Parent views | 📋 |
| **SCHEDULE MANAGEMENT** | | | | |
| Timetable | ✅ | ✅ Admin generates | ✅ Student/Teacher views own | 📋 |
| Teacher Allotment | ✅ | ✅ Admin assigns | — | — |
| Holidays | ✅ | ✅ Admin creates | ✅ Parent/Student views | — |
| Substitute | ✅ | ✅ Admin manages | — | — |
| **COMMUNICATION** | | | | |
| Announcements | ✅ | ✅ Admin creates | ✅ Parent/Student views | 📋 |
| Messages | ✅ | ✅ Staff send/receive | ✅ Parent/Student send/receive | 📋 |
| Notifications | ✅ | ✅ System generated | ✅ Parent/Student views | — |
| **COMPLAINTS** | | | | |
| Staff raises complaint on student | ✅ | ✅ Teacher raises | ✅ Parent views | — |
| Parent raises complaint | ✅ | ✅ Staff manages | ✅ Parent raises | — |
| Complaint resolution | ✅ | ✅ Staff resolves | ✅ Parent views resolution | — |
| **TRANSPORT** | | | | |
| Routes | ✅ | ✅ Admin manages | ✅ Parent views assigned route | 📋 |
| Vehicles | ✅ | ✅ Admin manages | ✅ Parent views assigned vehicle | 📋 |
| Drivers | ✅ | ✅ Admin manages | ✅ Parent views driver info | 📋 |
| Bus Tracking (live) | 🔲 | — | 🔲 Parent tracks live | 📋 |
| **STAFF MANAGEMENT** | | | | |
| Staff Profiles | ✅ | ✅ Admin manages | — | — |
| Staff Attendance (redesign) | ✅ 🔄 | ✅ 🔄 Pipeline + exception review | — | — |
| Leave Management | 🔲 | 🔲 Staff requests + admin oversight | — | — |
| Early Permissions | 🔲 | 🔲 Staff requests + monthly accumulation | — | — |
| Payroll (redesign) | ✅ 🔄 | ✅ 🔄 Workspace + LOP + advances | — | — |
| Roles & Departments | ✅ | ✅ Admin manages | — | — |
| **SCHOOL MANAGEMENT** | | | | |
| School Info | ✅ | ✅ Admin manages | ✅ Basic info shown | — |
| School Settings | ✅ | ✅ Admin manages | — | — |
| **CLASS SESSION** | | | | |
| Sessions | ✅ | ✅ Teacher manages | ✅ Student views | — |
| Class Progress | ✅ | ✅ Teacher records | ✅ Student/Parent views | — |
| **DASHBOARD** | | | | |
| Customisable Dashboard | — | ✅ Widget-based | — | — |
| **KYC-EXCLUSIVE FEATURES** | | | | |
| Diary | ✅ | — | ✅ Parent/Student writes | 📋 |
| Fees & Payments | 🔲 | 🔲 Finance mgmt (TBD) | 🔲 Parent pays | 📋 |
| Online Orders | 🔲 | 🔲 Order mgmt (TBD) | 🔲 Parent orders | 📋 |
| Library | 🔲 | 🔲 Library mgmt (TBD) | 🔲 Student browses | 📋 |
| Reminders | 🔲 | — | 🔲 System sends | — |
| **AUTHENTICATION** | | | | |
| Staff Login | ✅ | ✅ | — | — |
| Parent/Student Login | 🔲 Planned | — | 🔲 Planned | — |
| Multi-tenant (school ID) | ✅ | ✅ | ✅ | — |

---

## Website Content Coverage

The website describes features from both `sms-ui` and `kyc`. The features marked 📋 above are referenced in `website/content/website-content.md`.

**Keep this in sync:** When a feature moves from 🔲 Planned to ✅ Built, or when a feature is removed, update the website content doc and notify the content team to update the public website copy.

---

## New Navigation Entries (Sprints 1–8)

These new navigation items and profile tabs are being added as part of the current sprint sequence:

| New Entry | Location | Sprint |
|---|---|---|
| Setup (isolated nav) | Main navigation → Setup | Sprint 2 |
| Setup → Attendance → Shift Templates | Setup sub-screen | Sprint 2 |
| Setup → Attendance → Sources → Geofence | Setup sub-screen | Sprint 2 |
| Setup → Leave → Leave Types | Setup sub-screen | Sprint 5 |
| Setup → Leave → Early Permission Config | Setup sub-screen | Sprint 5 |
| My Profile → Leaves & Permissions tab | Profile tab (app) | Sprint 5 |
| My Profile → Approvals tab | Profile tab (app) | Sprint 5 |
| My Profile → Payslips tab | Profile tab (app) | Sprint 8 |
| Staff Management → Leave | Admin web screen | Sprint 5 |
| Staff Management → Staff Profile → Approvals tab | Admin web audit tab | Sprint 5 |
| Payroll → Settings tab | Payroll sub-screen | Sprint 8 |
| Payroll → Salary Templates tab | Payroll sub-screen | Sprint 8 |
| Payroll → Advances tab | Payroll sub-screen | Sprint 8 |

---

## Features with sms-ui Management Gap

These kyc features are built or planned but currently have no corresponding management view in sms-ui. As they grow, a sms-ui management module may be needed.

| kyc Feature | Status | sms-ui Gap |
|---|---|---|
| Diary | ✅ Built in kyc | No sms-ui view for staff |
| Fees & Payments | 🔲 Planned | Finance management module needed in sms-ui |
| Online Orders | 🔲 Planned | Order management module needed in sms-ui |
| Library | 🔲 Planned | Library management module needed in sms-ui |

---

## Related Documents

- `architecture/cross-project-dependencies.md` — dependency rules and impact guidance
- `architecture/data-flow.md` — per-module data flow direction between projects
- `sms-ui/modules/built/` — sms-ui module documentation
- `kyc/specs/built/` — kyc feature specifications
