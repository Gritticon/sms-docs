---
title: KYC Module Linkages Index
type: architecture
project: kyc
last-updated: 2026-03-18
---

# KYC — Module Linkages Index

This folder contains cross-module linkage documents for the KYC app. KYC modules are mostly read-only consumers of data sourced from sms-ui via sms-api. The key linkages are around what feeds into the Student Profile aggregator and what triggers Notifications.

## How to Use

**To find what triggers a notification:**
```
grep -r "Trigger:" docs/kyc/architecture/module-linkages/notifications-linkages.md
```

**To find what feeds into the student profile:**
```
grep -r "Depends on" docs/kyc/architecture/module-linkages/student-profile-linkages.md
```

**To trace a data change from sms-ui to KYC:**
Start at `architecture/data-flow.md`, find the module row, then follow to the relevant KYC linkage doc.

---

## Module Linkage Files

| File | Module | Depends On | Affects |
|---|---|---|---|
| `student-profile-linkages.md` | Student Profile | `Attendance`, `Timetable`, `Marks`, `Documents`, `Achievements`, `Certificates`, `Diary`, `Complaints`, `Requests`, `Class Session` | _(read-only aggregator)_ |
| `notifications-linkages.md` | Notifications | `Announcements`, `Attendance`, `Marks`, `Complaints`, `Requests`, `Fees (planned)`, `Reminders (planned)` | `Notification badge count (app-wide)` |

---

## Data Origin Note

All KYC module data (except Diary and parent-raised items like Complaints and Requests) originates in sms-ui and is served through sms-api. KYC does not have its own independent data stores for academic, attendance, or schedule data.

For cross-project data flow details see `architecture/data-flow.md`.
