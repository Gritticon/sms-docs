---
title: Temporal Data Model
type: architecture
project: platform
last-updated: 2026-03-19
---

# Temporal Data Model

This document defines the three-state data model used across the SMS platform. Every piece of data in the system belongs to one of three states based on its date relative to today. This model governs how data is stored, edited, displayed, and locked across all modules.

---

## The Three States

```
─────────────────────────────────────────────────────────────────────
  PAST                  │  TODAY               │  FUTURE
  Transaction           │  Staged              │  Settings-based
                        │                      │
  Immutable.            │  Live. Freely        │  Derived. No real
  Hard locked.          │  editable by anyone  │  data yet — projected
  Cannot be changed.    │  with access.        │  from configuration.
─────────────────────────────────────────────────────────────────────
        ◄───────────────┼──────────────────────┼───────────────►
                     yesterday              tomorrow
```

---

## State Definitions

### 1. Transaction (Past)

Any data whose date is **before today** is a transaction. It represents something that happened and is now a permanent record.

**Rules:**
- **Hard locked** — no edits permitted by any user
- Read-only for all roles including admin
- Displayed as historical fact in all views
- _(Planned)_ Admin-only amendment will be introduced later, gated by `manage` permission and requiring a mandatory audit log entry for every change

**Applies to:** Attendance records, completed exam results, finalized payroll, completed class sessions, past timetable slots, resolved complaints, past assignment submissions.

---

### 2. Staged (Current Day)

Any data whose date **equals today** is staged. It is live, in-progress, and has not yet been committed.

**Rules:**
- Freely editable by anyone with access to that module — no special permission required for edits
- When new data arrives for today, it **replaces or updates** the existing staged record for that date — there is no conflict, the latest write wins
- Automatically becomes a **transaction at midnight** — once the day rolls over, staged data is committed and hard locked as a transaction
- No manual submission or approval step — the date change is the trigger

**Applies to:** Today's attendance (staff, student, class), today's active timetable slots (including substitutes), today's class sessions, current month's payroll (if within the current month).

---

### 3. Settings-based (Future)

Any data whose date is **after today** is not real data — it is a projection derived from current configuration and settings. No record has been written yet.

**Rules:**
- Future data cannot be edited directly — to change future data, change the **setting** that generates it
- If the setting has an **effective date**, the change applies from that date forward
- If **no effective date is set**, the change applies immediately from the point the setting is saved, affecting all future dates
- Past effective dates are not allowed — settings cannot retroactively change transactions

**Applies to:** Future timetable slots (from timetable configuration), scheduled exams (from exam schedule settings), upcoming holidays (from school calendar settings), future payroll cycles (from salary and deduction settings), future class sessions (from timetable), upcoming assignment due dates (from assignment creation).

---

## Transition Rules

### Staged → Transaction
- **Trigger:** Midnight (date changes from today to yesterday)
- **Process:** Automatic — no user action required
- **Result:** The record is hard locked and becomes part of the permanent historical record

### Settings → Staged
- **Trigger:** The effective date of a setting reaches today
- **Process:** Automatic — the system resolves today's data from the active settings at the start of the day
- **Result:** A staged record is available for the current day based on the setting

### Settings change with effective date
```
Setting saved on: March 19
Effective date set to: April 1

→ March 19–31: Old setting still drives future projections
→ April 1 onwards: New setting drives future projections
→ Past data (before March 19): Unaffected — locked as transactions
```

### Settings change with no effective date
```
Setting saved on: March 19 (no effective date set)

→ From March 20 onwards: New setting drives all future projections
→ Today (March 19) if already staged: Unaffected — today is already staged
→ Past data: Unaffected — locked as transactions
```

---

## Module-by-Module Application

### Staff Attendance

| Date | State | Edit Rules |
|---|---|---|
| Before today | Transaction | Hard locked |
| Today | Staged | Freely editable; new mark replaces existing |
| Future | N/A | Attendance cannot be pre-marked |

---

### Student Attendance

| Date | State | Edit Rules |
|---|---|---|
| Before today | Transaction | Hard locked |
| Today | Staged | Freely editable; new mark replaces existing |
| Future | N/A | Attendance cannot be pre-marked |

---

### Class Attendance

| Date | State | Edit Rules |
|---|---|---|
| Before today | Transaction | Hard locked |
| Today | Staged | Freely editable per class/subject session |
| Future | N/A | Cannot be pre-marked |

---

### Timetable

| Date | State | Edit Rules |
|---|---|---|
| Before today | Transaction | Hard locked — historical record of scheduled slots |
| Today | Staged | Substitute assignments can be applied; display reflects override for today |
| Future | Settings-based | Derived from timetable configuration; change the timetable settings to change future slots |

**Effective date rule:** When a new timetable is generated, set an effective start date. Slots before that date remain as transactions or existing staged data. Slots from the effective date onwards derive from the new configuration.

---

### Exams

| Date | State | Edit Rules |
|---|---|---|
| Before today (completed) | Transaction | Exam details and results hard locked |
| Today | Staged | Exam details still editable |
| Future | Settings-based | Derived from exam schedule settings |

**Results specifically:** Results entry is staged on the day of entry. Once entered for a past exam date, they become a transaction.

---

### Payroll

| Period | State | Edit Rules |
|---|---|---|
| Past months | Transaction | Hard locked |
| Current month | Staged | Payroll figures can be adjusted until month end |
| Future months | Settings-based | Derived from salary and deduction settings |

**Effective date rule for salary settings:** If a salary change has an effective date of next month, the current month's staged payroll is unaffected. The change takes effect from the set date.

---

### Class Sessions

| Date | State | Edit Rules |
|---|---|---|
| Before today | Transaction | Session records and progress notes hard locked |
| Today | Staged | Teacher can update session details and class progress |
| Future | Settings-based | Derived from timetable configuration |

---

### Assignments

| Due Date | State | Edit Rules |
|---|---|---|
| Before today | Transaction | Assignment and submission records hard locked |
| Today | Staged | Assignment details editable; submission window open |
| Future | Settings-based | Created by teacher with a future due date; can be edited until due date arrives |

---

### Announcements

| Published Date | State | Edit Rules |
|---|---|---|
| Before today | Transaction | Published announcements hard locked |
| Today | Staged | Can be edited or retracted |
| Future | Settings-based | Scheduled announcements are settings — date determines when they become staged |

---

### Complaints

Complaints follow the same temporal model but always land in the **Transaction** state immediately — there is no staged phase.

| Action | State | Edit Rules |
|---|---|---|
| Complaint raised (by parent in KYC or teacher in sms-ui) | Transaction | Hard locked — cannot be edited by anyone |
| Status update (under investigation, resolved, etc.) | Transaction | Each status change is a new transaction entry appended to the record |

- Every complaint record, from the moment it is created, is a permanent transaction
- Neither the parent nor the teacher can edit a complaint once submitted
- Status changes do not modify the original record — they append a new transaction entry
- Complaints are never staged — there is no "today only" window for edits

---

### Staff and Student Profiles

Profile records are **master data**, not temporal. They do not follow the temporal model.

- All profile changes are committed immediately as transactions
- Future audit logging (planned) will record before/after values for all profile changes

---

### School Settings and Configuration

Settings are **always future-facing**. They do not hold historical state themselves — the transactions they generate hold the history.

- Changing a setting with an effective date affects data from that date forward
- Changing a setting without an effective date affects data from the point of save forward
- The setting change itself is logged (when it was changed and by whom)

---

## How This Affects KYC

Parents and students in the KYC app are always **read-only consumers** of this data. The temporal model affects what they see:

| State | What parent/student sees in KYC |
|---|---|
| Transaction (past) | Accurate historical records — attendance history, past results, completed sessions |
| Staged (today) | Live current-day data — today's attendance, active sessions, today's timetable |
| Settings-based (future) | Projected data — upcoming timetable, scheduled exams, upcoming holidays |

**Important:** If a staged record is updated multiple times today (e.g., attendance corrected), the parent sees the **latest version** in real time. There is no "pending" or "submitted" state visible to the parent — today's data is live.

---

## How This Affects Audit Logging

| State | Audit Logging |
|---|---|
| Staged edits (today) | Not audited — free editing, latest write wins |
| Transaction (hard locked) | No changes possible — nothing to audit |
| _(Planned)_ Admin amendment of transaction | Mandatory audit log entry required — records who changed what, before and after values, and reason |

---

## Open Items

| Item | Decision |
|---|---|
| Admin amendment of transactions | Planned — not yet implemented. Will require `manage` permission + mandatory audit entry |
| Payroll month-end staging cutoff | Is month-end automatic (last day of month) or does payroll admin manually close the period? |
| Announcement scheduled publishing | Are scheduled future announcements treated as settings until publish date, or as a separate draft state? |
