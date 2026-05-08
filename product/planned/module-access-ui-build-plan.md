---
title: Module Access UI — Cursor Build Plan
type: build-plan
project: platform
status: ready-to-build
last-updated: 2026-04-15
---

# Module Access UI — Cursor Build Plan

## What we are building

The Gritticon admin panel needs a UI to lock/unlock modules for each school based on their pricing plan. When a Gritticon admin changes a school's module access, the sms-ui (staff-facing Flutter app) must reflect those changes — hiding or showing modules from the navigation.

The backend logic (data model, schema, service) is already built. What is missing is the wiring: the API route, the admin panel tab, and the sms-ui integration.

**Full product specification:** `docs/product/planned/school-module-access-control.md`

---

## State of the Codebase Right Now

### Already built — sms-api
| File | Status |
|---|---|
| `app/models/school_module_access.py` | Done — ORM model, table `school_module_access` |
| `app/schemas/school_module_access_schema.py` | Done — Pydantic schemas |
| `app/services/school_module_access_service.py` | Done — business logic |
| `app/core/module_constants.py` | Done — `CORE_MODULE_IDS`, `CORE_SUBMODULE_IDS`, `ALL_MODULE_IDS` |

### Missing — sms-api
- Route file for the module access endpoints (does not exist yet)
- Registration of those routes in `app/main.py`

### Already built — admin-panel (`admin-panel/`)
- Stack: React 18 + TypeScript + Vite + Tailwind + shadcn/ui
- `src/pages/SchoolDetailPage.tsx` — school detail page (no Module Access tab yet)
- `src/App.tsx` — router (route `/schools/:id` maps to `SchoolDetailPage`)
- `src/lib/api.ts` — API utility with `api.get`, `api.post`, `api.put`
- `types/index.ts` — shared TypeScript types
- Components: `components/ui/` (shadcn), `components/schools/`, `components/layout/`

### Missing — admin-panel
- Types for module access in `types/index.ts`
- Module Access tab inside `SchoolDetailPage.tsx`
- A new component `ModuleAccessTab.tsx`

### Already built — sms-ui (`sms-ui/`)
- Stack: Flutter Web (GetX state management)
- `lib/config/module_config.dart` — canonical module/submodule definitions (IDs, titles, slugs)
- `lib/services/session_service.dart` — builds user session after login
- `lib/services/permission_service.dart` — filters navigation by RBAC permissions
- `lib/repositories/navigation_repository.dart` — builds sidebar nav items
- `lib/repositories/auth_repository.dart` — login/logout/token refresh

### Missing — sms-ui
- `lib/services/school_module_access_service.dart` — new singleton service
- Call to `GET /schools/{school_id}/module-access` in `session_service.dart` after login
- Navigation filtering that respects module access (on top of RBAC)
- "Module not available" screen for direct URL access to disabled modules

---

## Critical Technical Notes

### 1. Database is synchronous — service must match

The project uses **synchronous SQLAlchemy** (`Session`, not `AsyncSession`). See `app/core/database.py` — there is no async engine.

The existing `school_module_access_service.py` was written with `AsyncSession` and `await` calls — **this is wrong for this codebase**. The route and service must use the sync pattern consistent with all other routes.

**Pattern used everywhere else:**
```python
from sqlalchemy.orm import Session
from app.core.database import get_db

@router.get("/...")
async def my_route(db: Session = Depends(get_db)):
    result = db.query(MyModel).filter(...).all()  # sync, no await
```

When building the route file, rewrite the service calls to use sync SQLAlchemy (`.query()`, `.execute()`, `.commit()` without `await`). Do not use `AsyncSession`.

### 2. Auth patterns — two separate auth systems

Internal admin routes use internal employee JWT:
```python
from app.core.security import internal_employee_bearer
from app.middleware.auth_middleware import get_internal_employee_context, RequestContext

_token: None = Security(internal_employee_bearer),
auth: RequestContext = Depends(get_internal_employee_context)
# auth.user_id → internal employee ID
```

School-facing routes use staff JWT:
```python
from app.core.security import staff_bearer
from app.middleware.auth_middleware import get_staff_auth_context, RequestContext

_token: None = Security(staff_bearer),
auth: RequestContext = Depends(get_staff_auth_context)
# auth.school_id → school ID from JWT
```

Both are in `app/api/routes/school.py` as reference — study that file for exact import patterns.

### 3. Admin panel API base URL

In `admin-panel/src/lib/api.ts`:
- Dev: Vite proxy sends `/schools/...` → FastAPI
- The `api` util object has `api.get<T>(path)`, `api.put<T>(path, body)`
- Auth token is added automatically from local storage

### 4. Core modules are always ON — locked in UI

From `app/core/module_constants.py`:
```python
CORE_MODULE_IDS = frozenset({1, 4, 6, 7, 9, 13})
# Dashboard, Student Management, Staff Management, Class Management, School Management, Audit Logs

CORE_SUBMODULE_IDS = frozenset({301, 501, 507, 601, 605, 801, 802, 1301})
# Student Profiles, Staff Profiles, Roles & Departments, Classes & Sections, Subjects, School Info, School Settings, Audit Logs
```

These must render with a locked toggle in the admin panel (cannot be turned off). Their `is_enabled` is always `true` in the API response.

### 5. sms-ui school_id comes from session

After login, `session_service.dart` fetches `GET /schools/me` to get school info (stored in `GlobalService`). The school ID for the module access API call should be read from `GlobalService.school.id` (or from the JWT-embedded school_id in the token — check `StorageService` for how school_id is stored).

---

## Part 1 — sms-api: Create the Route File

### File to create: `app/api/routes/school_module_access.py`

Create two routers in this file:

**Router 1 — Internal admin endpoints** (prefix: `/internal/schools`, tag: `school-module-access`)

```
GET  /internal/schools/{school_id}/module-access
     Auth: internal_employee_bearer
     Response: SchoolModuleAccessResponse
     Returns: Full nested module config (all 14 modules, each with their submodules, is_enabled flags)
     
PUT  /internal/schools/{school_id}/module-access
     Auth: internal_employee_bearer
     Role check: role must be "super_admin" or "billing_manager" — support agents and viewers cannot write
     Body: SchoolModuleAccessUpdate { modules: [{ module_id, is_enabled, submodules: [{ submodule_id, is_enabled }] }] }
     Response: SchoolModuleAccessResponse (the new state after save)
```

**Router 2 — School-facing endpoint** (prefix: `/schools`, tag: `school-module-access`)

```
GET  /schools/{school_id}/module-access
     Auth: staff_bearer
     Guard: school_id in path must match auth.school_id from JWT (prevent cross-school reads)
     Response: SchoolModuleAccessSimple { enabled_modules: [int], enabled_submodules: [int] }
```

### Service adaptation (sync rewrite)

The existing `school_module_access_service.py` uses `AsyncSession` — adapt inline in the route file using sync SQLAlchemy. The core logic (module normalisation, core ID enforcement) stays the same. Key sync adaptations:

```python
# Async (existing service — do not call directly)
result = await db.execute(select(...))
rows = list(result.scalars().all())
await db.commit()

# Sync equivalent (use this in the route)
from sqlalchemy import select
result = db.execute(select(SchoolModuleAccess).where(...))
rows = list(result.scalars().all())
db.commit()
```

The helper functions `_rows_to_state`, `_build_response_from_state`, `_normalize_update_payload`, `_module_enabled`, `_effective_submodule_enabled` from the service file are pure Python — copy and use them directly (no async there, they're just data transforms).

### Role check for PUT

```python
def _require_write_access(auth: RequestContext):
    role = auth.staff_record.get("role")
    if role not in ("super_admin", "billing_manager"):
        raise HTTPException(status_code=403, detail="Only Super Admin or Billing Manager can edit module access")
```

### Wire into main.py

In `app/main.py`:

1. Add import at the top with other route imports:
```python
from app.api.routes import school_module_access
```

2. Add two `include_router` calls in the `api_router` block (after the existing `school.router` include):
```python
api_router.include_router(school_module_access.internal_router)
api_router.include_router(school_module_access.school_router)
```

---

## Part 2 — Admin Panel: Module Access Tab

### File to update: `types/index.ts`

Add these types at the end of the file:

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

### File to create: `components/schools/ModuleAccessTab.tsx`

This is the main UI component. Build it as follows:

#### Module catalog (client-side constant — mirrors `module_config.dart`)

Define a constant at the top of the file with the module/submodule names so the UI can display human-readable labels (the API only returns IDs):

```typescript
const CORE_MODULE_IDS = new Set([1, 4, 6, 7, 9, 13])
const CORE_SUBMODULE_IDS = new Set([301, 501, 507, 601, 605, 801, 802, 1301])

const MODULE_CATALOG = [
  { id: 1,  name: 'Dashboard',            submodules: [] },
  { id: 2,  name: 'Communication Hub',     submodules: [{ id: 101, name: 'Messages' }, { id: 102, name: 'Notifications' }, { id: 103, name: 'Announcements' }] },
  { id: 3,  name: 'Transport Management',  submodules: [{ id: 201, name: 'Routes' }, { id: 202, name: 'Vehicles' }, { id: 203, name: 'Drivers' }] },
  { id: 4,  name: 'Student Management',    submodules: [{ id: 301, name: 'Student Profiles' }, { id: 305, name: 'House Tag' }, { id: 306, name: 'Certificates' }, { id: 307, name: 'Documents' }, { id: 308, name: 'Achievements' }, { id: 309, name: 'Code of Conduct' }, { id: 311, name: 'Requests' }] },
  { id: 5,  name: 'Exam Management',       submodules: [{ id: 400, name: 'Exams' }, { id: 402, name: 'Question Papers' }, { id: 403, name: 'Results' }, { id: 404, name: 'Report Cards' }] },
  { id: 6,  name: 'Staff Management',      submodules: [{ id: 501, name: 'Staff Profiles' }, { id: 504, name: 'Attendance' }, { id: 505, name: 'Payroll' }, { id: 506, name: 'Leave Management' }, { id: 507, name: 'Roles & Departments' }] },
  { id: 7,  name: 'Class Management',      submodules: [{ id: 601, name: 'Classes & Sections' }, { id: 603, name: 'Assignments' }, { id: 604, name: 'Class Attendance' }, { id: 605, name: 'Subjects' }] },
  { id: 8,  name: 'Complaints',            submodules: [{ id: 701, name: 'View Complaints' }, { id: 702, name: 'Resolve Complaints' }] },
  { id: 9,  name: 'School Management',     submodules: [{ id: 801, name: 'School Info' }, { id: 802, name: 'School Settings' }] },
  { id: 10, name: 'Schedule Management',   submodules: [{ id: 602, name: 'Timetable' }, { id: 606, name: 'Teacher Allotment' }, { id: 607, name: 'Holidays' }, { id: 608, name: 'Substitute' }] },
  { id: 11, name: 'Watchlist',             submodules: [] },
  { id: 12, name: 'Class Session',         submodules: [{ id: 902, name: 'Class Progress' }] },
  { id: 13, name: 'Audit Logs',            submodules: [{ id: 1301, name: 'Audit Logs' }] },
  { id: 14, name: 'Fee Management',        submodules: [{ id: 1401, name: 'Fee Structures' }, { id: 1402, name: 'Fee Collection' }] },
]
```

#### Component props

```typescript
interface ModuleAccessTabProps {
  schoolId: number
  canEdit: boolean  // true for super_admin and billing_manager
}
```

#### State

- `config: SchoolModuleAccessResponse | null` — loaded from API
- `draft: SchoolModuleAccessResponse | null` — local edits before save
- `loading: boolean`
- `saving: boolean`
- `expandedModules: Set<number>` — which module cards are expanded
- `isDirty: boolean` — computed: draft differs from config

#### Data loading

On mount, call `GET /internal/schools/{schoolId}/module-access`. Store result in both `config` and `draft`.

#### Module card layout

For each module in `MODULE_CATALOG`, render a card:

**Header row (always visible):**
- Module name (left)
- If core: small grey "Core" badge + locked toggle (always on, disabled)
- If optional: toggle switch (on/off), controlled by `draft.modules.find(m => m.module_id === id).is_enabled`
- Submodule summary badge: "3 / 5 enabled" (count enabled non-core submodules out of total)
- Expand/collapse chevron (right)

**Toggle behaviour (optional modules only):**
- Toggle ON → set module `is_enabled = true`, set ALL submodules `is_enabled = true`
- Toggle OFF → set module `is_enabled = false`, set ALL non-core submodules `is_enabled = false` (core submodules stay true)

**Expanded body:**
- Each submodule as a row: submodule name (left) + toggle (right)
- Core submodules: grey "Core" label, toggle locked ON
- Optional submodules: toggle controlled by draft state
- Submodule toggle ON → also ensure parent module `is_enabled = true` (enabling a submodule enables the module)
- Submodule toggle OFF → if all non-core submodules are now OFF, set module `is_enabled = false`

#### Save footer

Sticky footer bar (only shown when `isDirty === true`):
- Left: "You have unsaved changes"
- Right: "Cancel" button (reset draft to config) + "Save Changes" button (calls PUT)

On save:
- Call `PUT /internal/schools/{schoolId}/module-access` with body `{ modules: draft.modules }`
- On success: update `config` to the API response, clear `isDirty`
- On error: show toast error

#### Unsaved changes guard

If user navigates away with `isDirty === true`, show a browser confirm dialog: "You have unsaved changes. Leave anyway?"

Use `useBeforeUnload` or the React Router `useBlocker` (if available in the version) to handle this.

#### Permissions

The `canEdit` prop is derived from the logged-in user's role (from `getAuth()` in `lib/auth.ts`). Read the role and pass it down:

```typescript
// in SchoolDetailPage.tsx
const auth = getAuth()
const canEdit = auth?.role === 'super_admin' || auth?.role === 'billing_manager'
```

If `canEdit === false`:
- All toggles are `disabled`
- Save footer is never shown
- A read-only notice is shown at the top: "You have view-only access to module configuration."

### File to update: `src/pages/SchoolDetailPage.tsx`

Add a tab system to the existing page. Currently the page is a flat layout — convert it to a tabbed layout with two tabs:

1. **Overview** — the existing school info, stats, and status management cards (unchanged)
2. **Module Access** — the new `<ModuleAccessTab>` component

Use shadcn `Tabs`, `TabsList`, `TabsTrigger`, `TabsContent` for the tab container.

Add `?tab=module-access` to the URL when the Module Access tab is active (use `useSearchParams`) so the tab state is shareable via URL.

Import and render `ModuleAccessTab` in the Module Access tab:
```tsx
import { ModuleAccessTab } from '@/components/schools/ModuleAccessTab'

// inside JSX:
<TabsContent value="module-access">
  <ModuleAccessTab schoolId={schoolId} canEdit={canEdit} />
</TabsContent>
```

---

## Part 3 — sms-ui: Module Access Integration

### File to create: `lib/services/school_module_access_service.dart`

```dart
/// Singleton service that stores the school's module access after login.
/// Call [loadModuleAccess] once after login. Use [isModuleEnabled] and
/// [isSubmoduleEnabled] throughout the app to gate navigation and screens.
class SchoolModuleAccessService extends GetxService {
  final Set<int> _enabledModules = {};
  final Set<int> _enabledSubmodules = {};
  bool _loaded = false;

  /// Load module access from the API. Called once after login.
  Future<void> loadModuleAccess(int schoolId) async { ... }

  /// Returns true if the module is enabled for this school.
  /// Core modules always return true even if the API hasn't loaded yet.
  bool isModuleEnabled(int moduleId) { ... }

  /// Returns true if the submodule is enabled for this school.
  bool isSubmoduleEnabled(int submoduleId) { ... }

  bool get isLoaded => _loaded;
}
```

**API call:** `GET /schools/{schoolId}/module-access`

Expected response shape:
```json
{
  "enabled_modules": [1, 4, 5, 6, 7, 9, 13],
  "enabled_submodules": [301, 501, 601, 605, 801, 802, 1301]
}
```

Parse into `_enabledModules` and `_enabledSubmodules` sets.

**Core module fallback:** If the API call fails, fallback to core-only set:
```dart
static const _coreModuleIds = {1, 4, 6, 7, 9, 13};
static const _coreSubmoduleIds = {301, 501, 507, 601, 605, 801, 802, 1301};
```

**GetX registration:** Register as permanent in the app's binding or at first use.

### File to update: `lib/services/session_service.dart`

In `initializeSession()` (login flow), after the user session is built and `PermissionService` is registered, add:

```dart
// After Get.put(PermissionService(userSession), permanent: true);

// Load school module access
final schoolId = globalService.school?.id;  // get from GlobalService after /schools/me is called
if (schoolId != null) {
  if (Get.isRegistered<SchoolModuleAccessService>()) {
    Get.delete<SchoolModuleAccessService>(force: true);
  }
  final moduleAccessService = Get.put(SchoolModuleAccessService(), permanent: true);
  try {
    await moduleAccessService.loadModuleAccess(schoolId);
  } catch (e) {
    // Silently fail — core modules will still be accessible
  }
}
```

Do the same in `refreshSession()` after the same point.

**Note:** The `/schools/me` call happens before this in `initializeSession` (when `name != null && email != null`). Get the school ID from `globalService.school?.id` after that call.

### File to update: `lib/repositories/navigation_repository.dart`

In `getNavigationItems()`, add a second filter pass after the RBAC permission filter. After the existing `permissionService.filterNavigationItems(allItems)` call, apply module access filtering:

```dart
List<NavigationItem> getNavigationItems() {
  final allItems = _getAllNavigationItems();
  
  // Step 1: RBAC permission filter (existing)
  List<NavigationItem> items = allItems;
  if (Get.isRegistered<PermissionService>()) {
    try {
      items = Get.find<PermissionService>().filterNavigationItems(allItems);
    } catch (e) {
      items = allItems;
    }
  }

  // Step 2: School module access filter (new)
  if (Get.isRegistered<SchoolModuleAccessService>()) {
    final moduleAccess = Get.find<SchoolModuleAccessService>();
    if (moduleAccess.isLoaded) {
      items = items.where((item) {
        return moduleAccess.isModuleEnabled(item.id);
      }).map((item) {
        if (item.submodules == null || item.submodules!.isEmpty) return item;
        final accessibleSubmodules = item.submodules!
            .where((sub) => moduleAccess.isSubmoduleEnabled(sub.id))
            .toList();
        if (accessibleSubmodules.isEmpty) return null;
        return NavigationItem(id: item.id, title: item.title, submodules: accessibleSubmodules);
      }).whereType<NavigationItem>().toList();
    }
  }
  
  return items;
}
```

### Direct URL access guard

In the app router (`lib/core/routing/app_router.dart`), for each module's route, add a guard that checks `SchoolModuleAccessService.isModuleEnabled(moduleId)`. If not enabled, redirect to a "Module Not Available" screen.

Create `lib/views/module_not_available_view.dart`:
- Message: "This module is not part of your school's current plan."
- No action button needed — staff cannot do anything about it. Just a back button.
- Do NOT show error styling — this is an expected state, not an error.

---

## Build Order

Follow this order — each step is independently testable:

```
Step 1 — sms-api route file (no UI changes needed to verify — test via /docs)
  1a. Create app/api/routes/school_module_access.py (sync SQLAlchemy, both routers)
  1b. Wire into app/main.py
  1c. Verify: GET /internal/schools/1/module-access returns all 14 modules
  1d. Verify: PUT /internal/schools/1/module-access updates correctly
  1e. Verify: GET /schools/1/module-access returns enabled_modules/enabled_submodules

Step 2 — admin-panel types and ModuleAccessTab (read-only first)
  2a. Add types to types/index.ts
  2b. Create components/schools/ModuleAccessTab.tsx (GET only, no editing yet)
  2c. Add tab to SchoolDetailPage.tsx
  2d. Verify: tab loads and shows all modules with correct enabled/disabled state

Step 3 — admin-panel editing
  3a. Enable toggle controls in ModuleAccessTab (PUT on save)
  3b. Add sticky save footer with dirty-state tracking
  3c. Add unsaved-changes guard
  3d. Verify: toggle a module, save, reload — state persists

Step 4 — sms-ui SchoolModuleAccessService
  4a. Create lib/services/school_module_access_service.dart
  4b. Call loadModuleAccess in session_service.dart (initializeSession + refreshSession)
  4c. Verify: after login, service contains correct enabled module IDs

Step 5 — sms-ui navigation filtering
  5a. Update navigation_repository.dart to filter by module access
  5b. Verify: disabling Fee Management in admin panel → Fee Management disappears from sms-ui nav after re-login

Step 6 — sms-ui direct URL guard
  6a. Create module_not_available_view.dart
  6b. Add route guards in app_router.dart
```

---

## Key Constraints — Do Not Violate

1. **No async DB in sms-api.** The DB layer is synchronous SQLAlchemy. Do not introduce `AsyncSession` or `asyncio` patterns in the route file.

2. **Core modules are immutable.** The API must never allow disabling them. The `_normalize_update_payload` logic in the service handles this — preserve that logic when adapting to sync.

3. **No data deletion on module disable.** Disabling a module in the admin panel does NOT delete any school data. It only hides the UI and blocks API access. The `set_school_module_access` service does a full replace of `school_module_access` rows — that is fine. It does NOT touch any school data tables.

4. **Staff are not told why.** In sms-ui, do not show "upgrade your plan" messages to staff. Simply hide the module. The `Module Not Available` screen (for direct URL access) explains it's a plan limitation but offers no action.

5. **No forced logout on access change.** When an admin changes module access, existing staff sessions are NOT invalidated. The change is enforced at the next API call (server-side 403) and at next page load (module disappears from navigation). No session invalidation needed.

6. **RBAC is unchanged.** The module access check is a pre-filter, not a replacement for role permissions. Both must pass for a staff member to access a module. Implementation order: module access check first, then RBAC check.

---

## Files Summary

### Create (new files)
| File | Project |
|---|---|
| `app/api/routes/school_module_access.py` | sms-api |
| `components/schools/ModuleAccessTab.tsx` | admin-panel |
| `lib/services/school_module_access_service.dart` | sms-ui |
| `lib/views/module_not_available_view.dart` | sms-ui |

### Modify (existing files)
| File | Project | Change |
|---|---|---|
| `app/main.py` | sms-api | Import + include_router for both module access routers |
| `types/index.ts` | admin-panel | Add 4 new TypeScript interfaces |
| `src/pages/SchoolDetailPage.tsx` | admin-panel | Add tab system, import ModuleAccessTab |
| `lib/services/session_service.dart` | sms-ui | Call loadModuleAccess after login/refresh |
| `lib/repositories/navigation_repository.dart` | sms-ui | Add module access filter after RBAC filter |
| `lib/core/routing/app_router.dart` | sms-ui | Add module access guards to routes |
