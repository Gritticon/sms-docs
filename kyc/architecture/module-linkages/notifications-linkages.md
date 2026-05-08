---
title: KYC Notifications Cross-Module Linkages
type: architecture
project: kyc
module: Notifications & Events
last-updated: 2026-03-18
---

# KYC Notifications — Cross-Module Linkages

> **Depends on:** `Announcements`, `Attendance`, `Marks`, `Complaints`, `Requests`, `Fees (planned)`, `Reminders (planned)`
> **Affects:** `Notification badge count (app-wide)`

Notifications in KYC are system-generated alerts triggered by events in other modules. A notification is not created manually — it is a downstream side effect of an action taken elsewhere (either in sms-ui or within KYC itself). Every module that generates notifications is listed here as a dependency.

---

## Depends On

### `Announcements`
When a new school announcement is published in sms-ui, a notification is pushed to the relevant parents and students in KYC.

**Trigger:** New announcement published in sms-ui Communication Hub.
**Recipient:** All parents and students in the announcement's target audience.
**Impact:** If the announcements module changes its publishing behaviour, notification delivery for announcements must be verified.

---

### `Attendance`
An absence notification is sent to a parent when their child is marked absent for the day.

**Trigger:** Student marked absent in sms-ui Student Attendance.
**Recipient:** Parent of the absent student.
**Impact:** If attendance marking behaviour changes (e.g., batch marking, corrections), review whether absence notifications are triggered correctly and not sent for corrected records.

---

### `Marks`
A notification is sent to parents and students when new exam results are published.

**Trigger:** Results published in sms-ui Exam Management.
**Recipient:** Student and parent of that student.
**Impact:** If result publishing is a phased or batch process, confirm notifications are sent only once results are fully published — not on intermediate saves.

---

### `Complaints`
A notification is sent when a complaint status is updated — either a new complaint is raised by a teacher against a student (parent is notified) or a staff member resolves/responds to a parent-raised complaint.

**Trigger:** New complaint raised in sms-ui against student; complaint status updated in sms-ui.
**Recipient:** Parent of the student.
**Impact:** If complaint workflow changes (new statuses, resolution steps), review notification triggers to ensure they fire at the right steps.

---

### `Requests`
A notification is sent to the parent when a staff member responds to or updates the status of a request they raised.

**Trigger:** Request status updated in sms-ui.
**Recipient:** Parent who raised the request.
**Impact:** If the request workflow adds new status steps, ensure notifications are triggered for the statuses that matter to the parent.

---

### `Fees — planned`
When a fee payment is due or overdue, a reminder notification will be sent to the parent.

**Trigger:** Fee due date reached (planned feature).
**Recipient:** Parent.
**Status:** Planned — not yet implemented.

---

### `Reminders — planned`
School-defined reminders (e.g., event reminders, exam reminders) will be pushed as notifications.

**Trigger:** Reminder date/time reached (planned feature).
**Recipient:** Defined audience (all parents, specific class, etc.).
**Status:** Planned — not yet implemented.

---

## Affects

### `Notification Badge Count (app-wide)`
Every unread notification increments the notification badge count displayed throughout the KYC app. All modules that generate notifications contribute to this count.

**Impact:** If a notification is generated incorrectly (e.g., duplicate, wrong recipient), it inflates the badge count. Each notification source module must ensure it fires triggers accurately.

---

## Change Impact Guide

When making changes to any module that triggers notifications, check:

- [ ] Does the change alter when or how a triggering event fires? → Verify the notification is still sent at the right moment
- [ ] Does the change add a new triggering event? → Add it to this document and implement the notification
- [ ] Was an attendance correction made? → Verify absence notifications are not re-triggered for corrected records
- [ ] Were results re-published or updated? → Verify the notification is not sent again for results the parent already received
- [ ] Was a complaint workflow step added? → Determine if the new step should trigger a notification
- [ ] Was a request workflow step added? → Determine if the new step should trigger a notification to the parent

---

## Related Documents

- `kyc/architecture/module-linkages/student-profile-linkages.md`
- `kyc/specs/built/notifications-events-spec.md`
- `kyc/specs/built/communication-spec.md`
- `kyc/specs/built/complaints-spec.md`
- `architecture/data-flow.md`
