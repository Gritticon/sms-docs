# Cursor Prompt — School Module Access UI

---

## Your role

You are implementing a feature across three sub-projects in a monorepo. Read all referenced files before writing any code. Complete the work step by step, verifying each step before moving to the next. At the end, produce a structured completion report.

---

## Read these files first — in this order

Before writing a single line of code, read and understand all of these:

**Product specification (start here — this is the source of truth for behaviour):**
- `docs/product/planned/school-module-access-control.md`

**Build plan (engineering decisions, file list, constraints, build order):**
- `docs/product/planned/module-access-ui-build-plan.md`

**Existing backend logic (already built — understand before creating the route):**
- `sms-api/app/models/school_module_access.py`
- `sms-api/app/schemas/school_module_access_schema.py`
- `sms-api/app/services/school_module_access_service.py`
- `sms-api/app/core/module_constants.py`

**Existing route patterns (copy auth/DB patterns from these — do not invent new ones):**
- `sms-api/app/api/routes/school.py` — shows both `internal_employee_bearer` and `staff_bearer` patterns
- `sms-api/app/api/routes/internal_employees.py` — shows `require_super_admin` role check pattern
- `sms-api/app/core/database.py` — confirms DB is synchronous SQLAlchemy (no async)
- `sms-api/app/main.py` — shows how to register routers

**Existing admin panel code (match these patterns exactly):**
- `admin-panel/src/pages/SchoolDetailPage.tsx`
- `admin-panel/src/lib/api.ts`
- `admin-panel/types/index.ts`
- `admin-panel/src/App.tsx`
- `admin-panel/components/schools/SchoolStatusBadge.tsx` — example component for style reference

**Existing sms-ui code (understand session and navigation before modifying them):**
- `sms-ui/lib/services/session_service.dart` — where to hook in the new module access load
- `sms-ui/lib/repositories/navigation_repository.dart` — where to add the module access filter
- `sms-ui/lib/services/permission_service.dart` — existing RBAC filter (module access goes on top of this)
- `sms-ui/lib/config/module_config.dart` — canonical module/submodule IDs and names
- `sms-ui/lib/core/routing/app_router.dart` — where to add route guards
- `sms-ui/lib/repositories/auth_repository.dart` — login flow reference

---

## What you are building

The Gritticon platform lets schools subscribe to different modules. Gritticon admins need a UI to lock/unlock modules per school. When a module is disabled, school staff should not see it in their SMS-UI navigation.

**Three-part feature:**
1. **sms-api** — Two API endpoints (admin-facing GET/PUT, school-facing GET). The backend service is already written; you are only creating the route file and wiring it in.
2. **admin-panel** — A new "Module Access" tab on the School Detail page. Admins toggle modules on/off and save.
3. **sms-ui** — After login, fetch module access and filter the navigation sidebar to hide disabled modules.

---

## Non-negotiable constraints

Read these carefully. Violating any of them will break the codebase.

**1. The database is synchronous. Do not use async SQLAlchemy.**

`sms-api/app/core/database.py` uses a sync engine and `Session`. All existing routes use `db: Session = Depends(get_db)` with no `await` on DB calls. The existing `school_module_access_service.py` was mistakenly written with `AsyncSession` — **do not call it directly**. Instead, port the logic into the route file using sync SQLAlchemy. The helper functions in the service file (`_rows_to_state`, `_build_response_from_state`, `_normalize_update_payload`, `_module_enabled`, `_effective_submodule_enabled`) are pure Python with no async — copy and use them as-is.

**2. Two separate auth systems exist. Use the right one.**

- Internal admin routes → `internal_employee_bearer` + `get_internal_employee_context`
- School-facing routes → `staff_bearer` + `get_staff_auth_context`

See exact import and usage pattern in `sms-api/app/api/routes/school.py`.

**3. Core modules cannot be disabled. Enforce this server-side.**

`CORE_MODULE_IDS = {1, 4, 6, 7, 9, 13}` and `CORE_SUBMODULE_IDS = {301, 501, 507, 601, 605, 801, 802, 1301}` from `module_constants.py`. Even if a PUT request tries to disable them, the server must treat them as enabled. The normalisation logic in the service already handles this — preserve it.

**4. No data is deleted when a module is disabled.**

Disabling a module only updates `school_module_access` rows. It does not touch any school data tables (students, staff, attendance, etc.).

**5. No forced logout when access changes.**

When an admin disables a module, existing staff sessions are not invalidated. The change is enforced at the next API call (server returns 403) and reflected at next login/reload.

**6. Staff should not see "upgrade" prompts.**

In sms-ui, simply hide disabled modules — no "purchase this module" messaging. For direct URL navigation to a disabled module, show a plain "This module is not available for your school" screen with only a back button.

---

## Step-by-step build instructions

Work through these steps sequentially. Do not start a step until the previous one is complete.

---

### Step 1 — sms-api: Create the route file

**File to create:** `sms-api/app/api/routes/school_module_access.py`

Create two routers in this file:

**`internal_router`** — prefix `/internal/schools`, tag `school-module-access`

```
GET  /internal/schools/{school_id}/module-access
     Auth: internal_employee_bearer + get_internal_employee_context
     Response: SchoolModuleAccessResponse
     Logic: Query school_module_access table for school_id, build full response
            using _rows_to_state and _build_response_from_state helper functions

PUT  /internal/schools/{school_id}/module-access
     Auth: internal_employee_bearer + get_internal_employee_context
     Role check: auth.staff_record.get("role") must be "super_admin" or "billing_manager"
                 → raise HTTPException(403) if not
     Body: SchoolModuleAccessUpdate
     Response: SchoolModuleAccessResponse
     Logic: Normalise payload (using _normalize_update_payload), delete existing rows
            for school_id, insert new rows, return updated state
            Record updated_by = auth.user_id on each new row
```

**`school_router`** — prefix `/schools`, tag `school-module-access`

```
GET  /schools/{school_id}/module-access
     Auth: staff_bearer + get_staff_auth_context
     Guard: if school_id != auth.school_id → raise HTTPException(403, "Forbidden")
     Response: SchoolModuleAccessSimple { enabled_modules: [int], enabled_submodules: [int] }
     Logic: Query rows, build full state, return only enabled IDs
            Core module/submodule IDs are always included in the response
```

All DB operations must be synchronous (`db.execute(...)`, `db.add(...)`, `db.commit()`).

---

### Step 2 — sms-api: Register the routes

**File to modify:** `sms-api/app/main.py`

Add to the imports block:
```python
from app.api.routes import school_module_access
```

Add after the `school.router` include in the `api_router` block:
```python
api_router.include_router(school_module_access.internal_router)
api_router.include_router(school_module_access.school_router)
```

**Verify Step 1–2:** Start the API server and confirm:
- `GET /internal/schools/1/module-access` returns all 14 modules with `is_enabled` flags
- `PUT /internal/schools/1/module-access` with a body toggling module 14 (Fee Management) saves correctly
- `GET /schools/1/module-access` returns `enabled_modules` and `enabled_submodules` arrays

---

### Step 3 — admin-panel: Add types

**File to modify:** `admin-panel/types/index.ts`

Append to the end of the file:

```typescript
export interface SubmoduleAccess {
  submodule_id: number
  is_enabled: boolean
}

export interface ModuleAccess {
  module_id: number
  is_enabled: boolean
  submodules: SubmoduleAccess[]
}

export interface SchoolModuleAccessResponse {
  school_id: number
  modules: ModuleAccess[]
}

export interface SchoolModuleAccessUpdate {
  modules: ModuleAccess[]
}
```

---

### Step 4 — admin-panel: Create ModuleAccessTab component

**File to create:** `admin-panel/components/schools/ModuleAccessTab.tsx`

Build this component to the following spec:

**Module catalog constant (at file top):**

Define `CORE_MODULE_IDS`, `CORE_SUBMODULE_IDS`, and `MODULE_CATALOG` as client-side constants that mirror the module IDs and names from `sms-ui/lib/config/module_config.dart`. These are needed to display human-readable names since the API returns only IDs.

Use these exact values:
```
Module 1  — Dashboard (no submodules)
Module 2  — Communication Hub → Messages (101), Notifications (102), Announcements (103)
Module 3  — Transport Management → Routes (201), Vehicles (202), Drivers (203)
Module 4  — Student Management → Student Profiles (301), House Tag (305), Certificates (306), Documents (307), Achievements (308), Code of Conduct (309), Requests (311)
Module 5  — Exam Management → Exams (400), Question Papers (402), Results (403), Report Cards (404)
Module 6  — Staff Management → Staff Profiles (501), Attendance (504), Payroll (505), Leave Management (506), Roles & Departments (507)
Module 7  — Class Management → Classes & Sections (601), Assignments (603), Class Attendance (604), Subjects (605)
Module 8  — Complaints → View Complaints (701), Resolve Complaints (702)
Module 9  — School Management → School Info (801), School Settings (802)
Module 10 — Schedule Management → Timetable (602), Teacher Allotment (606), Holidays (607), Substitute (608)
Module 11 — Watchlist (no submodules)
Module 12 — Class Session → Class Progress (902)
Module 13 — Audit Logs → Audit Logs (1301)
Module 14 — Fee Management → Fee Structures (1401), Fee Collection (1402)

Core module IDs: 1, 4, 6, 7, 9, 13
Core submodule IDs: 301, 501, 507, 601, 605, 801, 802, 1301
```

**Component props:**
```typescript
interface ModuleAccessTabProps {
  schoolId: number
  canEdit: boolean
}
```

**State management:**
- `config` — the saved state from the API (used to detect unsaved changes)
- `draft` — the working copy that toggles modify
- `expandedModules` — Set of module IDs currently expanded
- Loading and saving states

**On mount:** `GET /internal/schools/{schoolId}/module-access` → set both `config` and `draft`

**Module card (per module in MODULE_CATALOG):**

Collapsed view:
- Module name
- If core: grey "Core" badge + toggle locked ON (visually on, non-interactive)
- If optional and `canEdit`: interactive toggle
- If optional and `!canEdit`: toggle shown but `disabled`
- Submodule count badge: "X / Y enabled" (X = enabled non-core submodules, Y = total non-core submodules). Do not count core submodules in this badge.
- Expand chevron (right side)

Expanded view (submodule rows):
- Each submodule: name on left, toggle on right
- Core submodules: grey "Core" badge, toggle locked ON
- Optional submodules: controlled toggle (respects `canEdit`)

**Toggle logic:**
- Module toggle ON → set all submodules to `is_enabled: true` (core ones were already true)
- Module toggle OFF → set all non-core submodules to `is_enabled: false`
- Submodule toggle ON → also ensure parent module `is_enabled: true`
- Submodule toggle OFF → if all non-core submodules are now OFF, set parent module `is_enabled: false`

**Dirty state:** `isDirty = JSON.stringify(draft) !== JSON.stringify(config)` (or a proper deep-equal)

**Save footer (only when `isDirty && canEdit`):**
- Sticky bar at the bottom of the tab
- "You have unsaved changes" text + "Discard" button + "Save Changes" button
- On Save: `PUT /internal/schools/{schoolId}/module-access` with `{ modules: draft.modules }`
- On success: set `config` to API response, `draft` resets to match
- On error: show a toast error (use `sonner` — already installed)
- On Discard: reset `draft` to `config`

**Unsaved changes guard:** If `isDirty` when navigating away, show browser `confirm()` dialog.

**Read-only notice:** When `!canEdit`, show a muted info bar at the top of the tab: "You have view-only access to module configuration."

**Loading state:** While fetching, show a spinner centred in the tab area.

---

### Step 5 — admin-panel: Add Module Access tab to SchoolDetailPage

**File to modify:** `admin-panel/src/pages/SchoolDetailPage.tsx`

Wrap the existing page content in a tab system:

- Import shadcn `Tabs`, `TabsList`, `TabsTrigger`, `TabsContent` from `@/components/ui/tabs`
- Import `ModuleAccessTab` from `@/components/schools/ModuleAccessTab`
- Read `auth = getAuth()` and derive `canEdit = auth?.role === 'super_admin' || auth?.role === 'billing_manager'`
- Use `useSearchParams` to sync active tab to URL query param `?tab=overview` / `?tab=module-access`
- Default tab: `overview`

**Tab 1 — Overview:** Everything that currently exists on the page (school info card, stats card, status management card)

**Tab 2 — Module Access:** `<ModuleAccessTab schoolId={schoolId} canEdit={canEdit} />`

The page header (school name, School ID, status badge, back button) stays above the tabs — it is not inside any tab.

**Verify Steps 3–5:**
- Open a school detail page → two tabs visible
- Module Access tab loads all 14 modules correctly
- Core modules show locked toggles
- Toggling an optional module updates the UI instantly (draft state)
- Saving calls the PUT endpoint and persists
- Reloading the page shows the saved state
- Support agent / viewer sees toggles disabled

---

### Step 6 — sms-ui: Create SchoolModuleAccessService

**File to create:** `sms-ui/lib/services/school_module_access_service.dart`

```dart
import 'package:get/get.dart';
import '../config/module_config.dart';
import 'api_service.dart';

class SchoolModuleAccessService extends GetxService {
  static const _coreModuleIds = {1, 4, 6, 7, 9, 13};
  static const _coreSubmoduleIds = {301, 501, 507, 601, 605, 801, 802, 1301};

  final Set<int> _enabledModules = {..._coreModuleIds};
  final Set<int> _enabledSubmodules = {..._coreSubmoduleIds};
  bool _loaded = false;

  bool get isLoaded => _loaded;

  /// Call once after login. Fetches GET /schools/{schoolId}/module-access
  /// and populates enabled module/submodule sets.
  /// Falls back to core-only on any error.
  Future<void> loadModuleAccess(int schoolId) async {
    try {
      final apiService = Get.find<ApiService>();
      final response = await apiService.get('/schools/$schoolId/module-access');
      final data = response.data;

      final enabledModules = (data['enabled_modules'] as List?)
          ?.map((e) => e as int)
          .toSet() ?? _coreModuleIds;
      final enabledSubmodules = (data['enabled_submodules'] as List?)
          ?.map((e) => e as int)
          .toSet() ?? _coreSubmoduleIds;

      _enabledModules
        ..clear()
        ..addAll(enabledModules)
        ..addAll(_coreModuleIds);  // always include core regardless of API response
      _enabledSubmodules
        ..clear()
        ..addAll(enabledSubmodules)
        ..addAll(_coreSubmoduleIds);

      _loaded = true;
    } catch (_) {
      // Fallback: core only
      _enabledModules..clear()..addAll(_coreModuleIds);
      _enabledSubmodules..clear()..addAll(_coreSubmoduleIds);
      _loaded = true;
    }
  }

  bool isModuleEnabled(int moduleId) {
    if (_coreModuleIds.contains(moduleId)) return true;
    return _enabledModules.contains(moduleId);
  }

  bool isSubmoduleEnabled(int submoduleId) {
    if (_coreSubmoduleIds.contains(submoduleId)) return true;
    return _enabledSubmodules.contains(submoduleId);
  }
}
```

---

### Step 7 — sms-ui: Hook into session_service.dart

**File to modify:** `sms-ui/lib/services/session_service.dart`

In `initializeSession()`: after `Get.put(PermissionService(userSession), permanent: true)` and after the `/schools/me` call stores school data in `GlobalService`, add:

```dart
// Load school module access
try {
  final globalService = Get.find<GlobalService>();
  final schoolId = globalService.school?.id;
  if (schoolId != null) {
    if (Get.isRegistered<SchoolModuleAccessService>()) {
      Get.delete<SchoolModuleAccessService>(force: true);
    }
    final moduleAccessService = Get.put(SchoolModuleAccessService(), permanent: true);
    await moduleAccessService.loadModuleAccess(schoolId);
  }
} catch (_) {
  // Silently fail — core modules remain accessible
}
```

Apply the same addition in `refreshSession()` at the equivalent point.

Add the import: `import 'school_module_access_service.dart';`

---

### Step 8 — sms-ui: Add module access filter to navigation

**File to modify:** `sms-ui/lib/repositories/navigation_repository.dart`

In `getNavigationItems()`, after the RBAC filter (the existing `permissionService.filterNavigationItems(allItems)` call), add a second filter pass for module access:

```dart
// Module access filter — runs after RBAC filter
if (Get.isRegistered<SchoolModuleAccessService>()) {
  final moduleAccess = Get.find<SchoolModuleAccessService>();
  if (moduleAccess.isLoaded) {
    items = items
        .where((item) => moduleAccess.isModuleEnabled(item.id))
        .map((item) {
          if (item.submodules == null || item.submodules!.isEmpty) return item;
          final visibleSubs = item.submodules!
              .where((sub) => moduleAccess.isSubmoduleEnabled(sub.id))
              .toList();
          if (visibleSubs.isEmpty) return null;
          return NavigationItem(id: item.id, title: item.title, submodules: visibleSubs);
        })
        .whereType<NavigationItem>()
        .toList();
  }
}
```

Add import: `import 'school_module_access_service.dart';`

---

### Step 9 — sms-ui: Module not available screen + route guard

**File to create:** `sms-ui/lib/views/module_not_available_view.dart`

Simple screen:
- Title: "Module Not Available"
- Body: "This module is not part of your school's current plan. Please contact your school administrator."
- A back button (navigate to dashboard or pop route)
- Do not use error colours (red). Use neutral/muted styling — this is an expected state.

**File to modify:** `sms-ui/lib/core/routing/app_router.dart`

For each module route that is optional (non-core), add a guard that checks:
```dart
if (Get.isRegistered<SchoolModuleAccessService>()) {
  final svc = Get.find<SchoolModuleAccessService>();
  if (svc.isLoaded && !svc.isModuleEnabled(MODULE_ID)) {
    return ModuleNotAvailableView();
  }
}
```

Apply guards to: Communication Hub (2), Transport (3), Exam Management (5), Complaints (8), Schedule Management (10), Watchlist (11), Class Session (12), Fee Management (14). Do NOT add guards to core modules (1, 4, 6, 7, 9, 13).

---

## Completion report

When all steps are done, provide a report in this exact format:

```
## Module Access UI — Completion Report

### Files created
- [path] — [one-line description]
...

### Files modified
- [path] — [what changed]
...

### Steps completed
- [x] Step 1 — sms-api route file created
- [x] Step 2 — Registered in main.py
- [x] Step 3 — Admin panel types added
- [x] Step 4 — ModuleAccessTab component built
- [x] Step 5 — Tab added to SchoolDetailPage
- [x] Step 6 — SchoolModuleAccessService created
- [x] Step 7 — Session service updated
- [x] Step 8 — Navigation filter added
- [x] Step 9 — Module not available screen + route guards

### Known deviations from build plan
[List anything you could not implement exactly as specified, and why]

### What to manually verify
[List the key things the developer should click through to confirm correct behaviour]
```

Do not mark a step complete until you have verified it works as described. If you encounter an ambiguity not covered by the spec or build plan, state your assumption clearly in the report under "Known deviations".
