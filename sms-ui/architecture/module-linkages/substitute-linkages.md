---
title: Substitute Cross-Module Linkages
type: architecture
project: sms-ui
module: Schedule Management — Substitute
last-updated: 2026-03-18
---

# Substitute — Cross-Module Linkages

> **Depends on:** `Timetable`, `Staff Profiles`
> **Affects:** `Timetable (view — day-level override)`

The substitute module allows a staff member to temporarily cover a class in place of the regular teacher for a specific day. It does not alter the base timetable — it overrides the display for that day only. The base timetable remains intact and the regular teacher's schedule resumes when the substitute period ends.

---

## Depends On

### `Timetable`
A substitute can only be assigned to an existing timetable slot. The substitute module reads the current timetable to determine which slots are available for substitution on a given day.

**Impact:** If the timetable is regenerated or a slot is removed, any substitute assignments for that slot may become invalid and must be reviewed.

---

### `Staff Profiles`
Only active staff members can be assigned as substitutes. The substitute module pulls the list of available staff from Staff Profiles.

**Impact:** If a staff member assigned as a substitute is deactivated, their substitute assignment must be reassigned to another active staff member.

---

## Affects

### `Timetable — Day View (override only)`
When a substitute is active for a slot on a given day, the timetable view for that day shows the substitute teacher in place of the regular teacher. This is a display-level override — it does not modify the underlying timetable record.

**Scope of override:** The substitution is scoped to the specific timetable slot and day. Other slots and other days are not affected.

**When substitution ends:** Once the substitute period passes, the timetable view automatically reverts to the regular teacher for that slot. No manual reversion is needed.

**Impact:** Staff viewing the timetable on the substitution day will see the substitute teacher. The regular teacher's profile schedule also reflects the substitution for that day — their slot is shown as covered.

---

## Change Impact Guide

When making changes to the **Substitute** module, check:

- [ ] Is the substitute being assigned to a valid timetable slot? → Verify the slot exists in `Timetable`
- [ ] Is the substitute teacher an active staff member? → Verify in `Staff Profiles`
- [ ] Was the timetable regenerated after a substitute was assigned? → Check that the substitute assignment is still linked to a valid slot
- [ ] Was a substitute deactivated as a staff member mid-assignment? → Reassign the substitute slot immediately
- [ ] Is the substitution day-scoped correctly? → Confirm it does not affect slots on other days or affect the base timetable record

---

## Related Documents

- `sms-ui/architecture/module-linkages/timetable-linkages.md`
- `sms-ui/architecture/module-linkages/staff-profile-linkages.md`
