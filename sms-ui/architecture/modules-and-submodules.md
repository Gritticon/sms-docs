---
title: Modules and Submodules
type: architecture
project: sms-ui
last-updated: 2026-03-18
---

MODULES AND SUBMODULES
======================

Permission IDs (1–180) are assigned by the backend in sorted module id, then sorted submodule id per module; see sms/docs/BACKEND_PERMISSION_IMPLEMENTATION.md.

MODULES
-------
1  - Dashboard
2  - Communication Hub
3  - Transport Management
4  - Student Management
5  - Exam Management
6  - Staff Management
7  - Class Management
8  - Complaints
9  - School Management
10 - Schedule Management (Timetable)
12 - Class Session


SUBMODULES
----------

Module 1: Dashboard
  (No submodules)

Module 2: Communication Hub
  101 - Messages
  102 - Notifications
  103 - Announcements

Module 3: Transport Management
  201 - Routes
  202 - Vehicles
  203 - Drivers

Module 4: Student Management
  301 - Student Profiles
  305 - House Tag
  306 - Certificates
  307 - Documents
  308 - Achievements
  309 - Code of Conduct
  311 - Requests

Module 5: Exam Management
  400 - Exams (includes exam schedules; 401 reserved/deprecated)
  402 - Question Papers
  403 - Results
  404 - Report Cards

Module 6: Staff Management
  507 - Roles & Departments
  501 - Staff Profiles
  504 - Staff Attendance
  505 - Payroll

Module 7: Class Management
  601 - Classes & Sections
  605 - Subjects
  603 - Assignments
  604 - Class Attendance

Module 8: Complaints
  701 - View Complaints
  702 - Resolve Complaints

Module 9: School Management
  801 - School Info
  802 - School Settings

Module 10: Schedule Management (Timetable)
  602 - Timetable
  606 - Teacher Allotment
  607 - Holidays
  608 - Substitute

Module 12: Class Session
  901 - Sessions
  902 - Class Progress

