---
title: Roles & Departments Cross-Module Linkages
type: architecture
project: sms-ui
module: Staff Management — Roles & Departments
last-updated: 2026-03-18
---

# Roles & Departments — Cross-Module Linkages

> **Depends on:** _(none — foundational module)_
> **Affects:** `Staff Profiles`, `Permission System`, `Navigation`, `All Modules (indirectly)`

Roles & Departments is a foundational module with no upstream dependencies. It defines the organisational structure of the school's staff and controls access across the entire platform. Every other module is indirectly affected by it because permissions are role-based — what a staff member can see and do in sms-ui is determined entirely by their assigned role.

---

## Depends On

This module has no upstream module dependencies. It is the foundational layer.

---

## Affects

### `Staff Profiles`
Every staff member is assigned a role and belongs to a department via their profile. The profile displays this assignment and the role determines the staff member's access level.

**Impact:** Changing a staff member's role immediately changes their permission set. Deleting a role that has staff assigned to it must be handled carefully — staff must be reassigned to another role before deletion.

---

### `Permission System`
Each role holds a set of permission IDs (integers 1–180) that define which modules and actions the role can access. The permission system reads these IDs to determine what is shown and enabled for each staff member.

**Impact:** Adding or removing permission IDs from a role changes access for every staff member assigned to that role simultaneously. This is a broad, immediate change — review all staff assigned to the role before modifying permissions.

---

### `Navigation (sms-ui sidebar)`
The sidebar navigation is filtered at runtime based on the logged-in staff member's role permissions. Only modules the staff member has any permission for are shown.

**Impact:** Changing a role's permissions changes the navigation menu for all staff in that role on their next login or session refresh.

---

### `All Modules (indirectly)`
Because every action in every module is gated by permissions derived from the staff member's role, Roles & Departments indirectly affects the usability of every module. A staff member with insufficient role permissions cannot create, edit, delete, or manage records — even if they can navigate to the module.

**Impact:** When a new module or submodule is added to the system, the relevant permission IDs must be added to the appropriate roles. Otherwise staff in those roles will have no access.

---

## Change Impact Guide

When making changes to **Roles & Departments**, check:

- [ ] Was a role's permission IDs changed? → All staff in that role are immediately affected; verify no staff lose access they need; verify no staff gain access they should not have
- [ ] Was a new role created? → Assign the correct permission IDs before assigning any staff to it
- [ ] Was a role deleted? → Verify no staff are still assigned to it; reassign affected staff first
- [ ] Was a department changed or deleted? → Verify staff profile department assignments are updated
- [ ] Was a new module added to the system? → Add the new module's permission IDs to all roles that should have access; see `sms-api/architecture/permissions-adding-modules-checklist.md`
- [ ] Did navigation change unexpectedly for a staff member? → Check their role's permission IDs first

---

## Related Documents

- `sms-ui/architecture/module-linkages/staff-profile-linkages.md`
- `sms-api/architecture/permission-implementation.md`
- `sms-api/architecture/permissions-adding-modules-checklist.md`
- `sms-ui/architecture/modules-and-submodules.md`
