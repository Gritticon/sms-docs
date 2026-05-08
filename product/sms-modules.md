---
title: SMS Module Specifications
type: product
project: platform
last-updated: 2026-03-18
---

# SMS — Module Specifications

SMS (School Management System) is the staff-facing web application. It is used by all school staff — teaching and non-teaching — to manage, plan, and organise all school operations.

---

## Module Status Overview

| Module | Status |
|--------|--------|
| Staff Management | Complete |
| Student Management | Complete |
| Classes & Sections | Complete |
| Subjects | Complete |
| Academic Records | Complete |
| Exams & Question Papers | Complete |
| Roles & Departments | Complete |
| Payroll & Salary | Complete |
| Timetables | Complete |
| Holidays & Academic Calendar | Complete |
| Rooms & Subject Hours | Complete |
| Student Leaves | Complete |
| Student Documents & Certificates | Complete |
| Student Achievements & Code of Conduct | Complete |
| House Tags | Complete |
| Watchlist | Complete |
| Substitute / Cover | Complete |
| Session Logs | Complete |
| Staff Attendance | Complete |
| Teacher Diary | Pending |
| Dashboard | Pending |
| Communication Hub | Pending |
| Transport Management | Pending |
| Complaints | Pending |
| School Management | Pending |
| Fee Management | Pending |
| Inventory Management | Pending |
| Order Management | Pending |
| Audit Logging (dedicated module) | Pending |

---

## Completed Modules

### Staff Management
- Full CRUD for staff profiles
- Bulk operations and custom field filtering
- Role and department assignment
- Profile image upload

### Student Management
- Student profiles with admission status
- Transport mode, house tag assignment
- Tabs: Academic Records, Achievements, Certificates, Code of Conduct, Documents
- Dynamic fields

### Classes & Sections
- Create and manage classes and sections
- Assign subjects to classes
- Flexible naming and structure

### Subjects
- Subject catalogue
- Subject options (electives)
- Class-subject allocation

### Academic Records
- Student grades per subject per exam
- Term-based structure

### Exams & Question Papers
- Exam configuration and scheduling
- Question paper management per exam and subject
- Results publishing

### Roles & Departments
- Custom roles with permission matrix UI
- Department creation and assignment
- 180-permission RBAC system

### Payroll & Salary
- Salary templates
- Earning and deduction masters
- Monthly payroll runs
- Individual payslips

### Timetables
- Timetable generation with templates
- Period and break slot configuration
- Room and subject-hour allocation
- Preview and publishing

### Holidays & Academic Calendar
- Holiday management
- Academic year configuration
- Session tracking

### Rooms & Subject Hours
- Classroom resource management
- Subject time allocation per class

### Student Leaves
- Leave request submission and approval flow
- Status: Pending, Approved, Rejected, Cancelled

### Student Documents & Certificates
- Document upload and management
- Certificate generation

### Student Achievements & Code of Conduct
- Awards and achievement records
- Behavioural tracking

### House Tags
- House/team grouping for students
- Points and ranking system

### Watchlist
- Flag students for monitoring

### Substitute / Cover
- Staff substitution assignment

### Session Logs
- User session tracking

### Staff Attendance
- Attendance records with bucket dates
- Audit history displayed inline

---

## Pending Modules

### Teacher Diary
A module interlinked with Timetable and Student Attendance.

**Flow:**
1. Teacher logs in → their profile shows **today's timetable by default**
2. Teacher clicks on a subject period
3. If the school has enabled attendance in settings → teacher is prompted to **mark student attendance first** (class and section are auto-fetched from the timetable — no manual selection required)
4. After attendance → teacher gets the option to:
   - Add **homework** for that period
   - Make a **progress note** (what was covered, what is planned)

**Why this flow:** Eliminates the need for teachers to manually select class, section, and period — it is all derived from the timetable context.

**Feeds into KYC:** Homework and progress notes appear in the KYC Diary module for parents and students.

---

### Dashboard
The primary landing screen after login. Summarises key data relevant to the logged-in user's role.

Planned content (to be defined during build based on role):
- Attendance summary (today's, weekly)
- Pending approvals (leaves, complaints)
- Upcoming exams and events
- Fee collection status
- Quick actions

---

### Communication Hub
Central communication tool for school staff and management.

**Features:**
- **Announcements** — broadcast messages to staff, students, or parents
- **Internal messaging** — staff-to-staff messaging
- **Notifications to parents** — push updates to KYC
- **Posts** — create and manage picture posts of events or key notes that appear on the KYC home feed (Instagram-style)

---

### Transport Management
Manage school transport operations.

**Features:**
- Route management
- Bus / vehicle records
- Student transport assignment
- Driver and conductor records
- Transport fee linkage

---

### Complaints
Manage complaints submitted by staff, parents, or students.

**Features:**
- View and categorise incoming complaints
- Assign complaints to staff for resolution
- Status tracking (open, in progress, resolved)
- Response and communication trail

---

### School Management
School profile, settings, and configuration.

**Features:**
- School profile (name, logo, address, contact)
- General settings
- KYC Settings:
  - Set the school's brand color (used in KYC)
  - Show/hide KYC modules (schools can disable modules they don't use)
- Academic year configuration
- Attendance settings (enable/disable attendance marking per timetable)
- Payment settings (enable/disable online fee payments)

---

### Fee Management
Full fee lifecycle management.

**Features:**
- Fee structure creation (term-based, category-based)
- Student fee assignment
- Due date management
- Payment recording (manual and online via KYC)
- Overdue tracking
- Fee receipts and invoice generation
- Optional online payment gateway integration (opt-in per school)

---

### Inventory Management
Track and manage all school physical assets and consumables.

**Categories:**
- Stationery
- Lab equipment
- Library books
- School assets (furniture, electronics, etc.)

**Features:**
- Item catalogue with quantity tracking
- Stock-in / stock-out records
- Low stock alerts
- Student order fulfilment (linked to KYC Orders)
- Library book borrow/return tracking

---

### Order Management
Handles orders placed by students/parents via KYC.

**Features:**
- View all incoming orders
- Order status management (received, processing, ready, delivered)
- Delivery option handling (school pickup vs home delivery)
- Inventory deduction on order fulfilment
- Order history

---

### Audit Logging (Dedicated Module)
A standalone module providing a unified view of all audit activity across SMS.

**Context:** Audit logs are already being recorded and displayed inline in Staff Attendance and a few other modules as history. This module consolidates them.

**Features:**
- Unified audit log viewer
- Filter by: user, module, action, date range, school
- Export logs
- Useful for compliance, investigation, and accountability
