---
title: Adding New Modules or Submodules – Permission Checklist
type: process
project: sms-api
last-updated: 2026-03-18
---

# Adding New Modules or Submodules – Permission Checklist

This document prevents the error: **"Staff doesn't have view permission for module X submodule Y"** (and similar permission issues) when adding new modules or submodules. Follow this checklist every time you add or change a submodule.

---

## 1. Backend (sms-api)

### 1.1 Permission registry

**File:** `sms-api/app/core/permissions.py`

- Add the new submodule to the correct module in `MODULES`, e.g.  
  `(608, "Substitute")` under module `10`.
- **Ensure `MAX_PERMISSION_ID`** is at least the last assigned ID.  
  Each new submodule adds 5 IDs (view, create, edit, delete, manage).  
  Current range is 1–180 (36 submodules × 5). If you add a submodule, increment `MAX_PERMISSION_ID` by 5 (e.g. to 185 for one more submodule).

### 1.2 Admin permissions script (critical)

**File:** `sms-api/scripts/update_admin_permissions.py`

The script sets the Admin role’s permissions to **all** permission IDs from the registry:  
`ADMIN_PERMISSIONS = PermissionRegistry.get_all_permission_ids()` (1–180).

- **You do not add new IDs by hand.** When you add a new submodule in `permissions.py`, you must:
  1. Add the submodule to the correct module in `MODULES` and ensure `MAX_PERMISSION_ID` is updated so the new 5 IDs are within range (the cache assigns IDs sequentially; the script uses `get_all_permission_ids()`).
  2. **Re-run the script** so the Admin role is updated:  
     `python scripts/update_admin_permissions.py <school_id> [Admin]`  
     Run from the `sms-api` directory.

**Why view is required:**  
Read endpoints use `check_staff_permission(..., action="view")`. If the Admin role (by name) is not role_id 1, the API returns permissions **stored in the database**. The script assigns all 1–180 (including view for every submodule), so the Admin role can call all GET endpoints.

**How IDs are assigned:**  
IDs are assigned in order: sorted module id → sorted submodule id per module → actions (view, create, edit, delete, manage). The backend’s `PermissionRegistry._build_permission_cache()` defines this order; `get_all_permission_ids()` returns 1 through `MAX_PERMISSION_ID` (currently 180).

### 1.3 Routes

- Use `check_staff_permission(db, auth.school_id, auth.user_id, MODULE_ID, SUBMODULE_ID, action)` on every substitute (or new submodule) route.
- Use action `"view"` for GET endpoints and `"create"` / `"edit"` / `"delete"` / `"manage"` as appropriate for write endpoints.

### 1.4 After changing permissions

- Run the admin script so the Admin role has the new IDs:  
  `python scripts/update_admin_permissions.py <school_id> [Admin]`
- Users must log in again (or refresh session) to receive updated permissions.

---

## 2. Frontend (sms)

### 2.1 Module config

**File:** `sms/lib/config/module_config.dart`

- Add a constant for the new submodule, e.g.  
  `static const int submoduleSubstitute = 608;`
- Add `SubmoduleMetadata(id: submoduleSubstitute, title: 'Substitute', slug: 'substitute')` to the correct module’s `submodules` list (same order as backend for consistent permission IDs).

### 2.2 Permission gates in the view

- Use **view** for access to the screen (handled by navigation/session).
- Use **create** for assign/create actions (e.g. assign substitute, add button).
- Use **edit** for edit actions.
- Use **delete** for remove/delete actions (e.g. remove cover).
- Use **PermissionGate** (or equivalent) so that:
  - Users without **create** cannot see or use assign/create UI.
  - Users without **delete** cannot see or use delete/remove UI.

Example (create and delete):

```dart
PermissionGate(
  moduleId: ModuleConfig.moduleTimetable,
  submoduleId: ModuleConfig.submoduleSubstitute,
  action: 'create',  // or 'delete'
  child: YourButtonOrWidget(),
)
```

- For drag-and-drop assign, pass a `canAssign` (or similar) that is true only when not assigning **and** the user has create permission (e.g. from `PermissionService.canCreate(moduleId, submoduleId)`), and show a message when a slot is selected but the user lacks create permission.

### 2.3 Permission IDs on the frontend

- Permission IDs are derived from `ModuleConfig.moduleSubmodules` in `PermissionRegistry.getAllPermissions()` (`sms/lib/services/permission_registry.dart`).  
- Keep **module and submodule order** in `module_config.dart` **the same as** in `sms-api/app/core/permissions.py` so that permission IDs match between backend and frontend.

---

## 3. Quick checklist (copy for each new submodule)

| Step | Location | Action |
|------|----------|--------|
| 1 | `sms-api/app/core/permissions.py` | Add `(submodule_id, "Name")` to the module’s list; ensure `MAX_PERMISSION_ID` covers the new IDs (+5 per submodule). |
| 2 | `sms-api/scripts/update_admin_permissions.py` | No change needed; script uses `get_all_permission_ids()`. Re-run after any change to `permissions.py`. |
| 3 | `sms-api` routes | Use `check_staff_permission(..., submodule_id, "view")` for GET and appropriate action for POST/PUT/DELETE. |
| 4 | `sms/lib/config/module_config.dart` | Add submodule constant and `SubmoduleMetadata` in the same order as backend. |
| 5 | `sms` view | Add PermissionGate (or equivalent) for create/edit/delete as needed; gate assign/create and delete UI. |
| 6 | After deploy | Run `update_admin_permissions.py` for each school; users re-login or refresh session. |

---

## 4. Reference: Substitute (608) fix applied

- **Issue:** Admin saw "staff doesn't have view permission for module 10 submodule 608" because the Admin role (by name) had only create/edit/delete/manage stored and not **view** for Substitute (permission ID 166).
- **Fix:** The admin script was updated to use `PermissionRegistry.get_all_permission_ids()` (1–180) instead of a manual list, so the Admin role gets view, create, edit, delete, and manage for every submodule. Re-running the script updates the DB. `MAX_PERMISSION_ID` was set to 180 to match the cache (36 submodules × 5).
- **UI:** Create permission is used to allow/hide assign (drop) and to show a message when the user lacks create permission.

---

*Last updated: permission script uses get_all_permission_ids(); MAX_PERMISSION_ID 180; 36 submodules.*
