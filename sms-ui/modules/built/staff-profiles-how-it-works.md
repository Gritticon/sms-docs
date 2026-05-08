---
title: Staff Profile — How It Works
type: module
project: sms-ui
module: Staff Management
last-updated: 2026-03-18
---

# Staff Profile – How It Works

Document for developers: API triggers, frontend/backend flow, and grep-friendly reference.

---

## GREP QUICK REFERENCE

| Grep for | Meaning |
|----------|--------|
| `STAFF_PROFILE_API` | All staff-profile–related API paths |
| `STAFF_PROFILE_FRONTEND` | Flutter entry points and flow |
| `STAFF_PROFILE_BACKEND` | FastAPI routes and services |
| `STAFF_PROFILE_PERMISSION` | Permission/module IDs |
| `API_GET` / `API_POST` / `API_PUT` / `API_DELETE` | HTTP method used |

---

## 1. STAFF_PROFILE_API – Endpoints Used by Staff Profiles

### 1.1 List staff (main table data)

- **API_GET** `GET /staff/`
- **STAFF_PROFILE_BACKEND** Route: `sms-api/app/api/routes/staff.py` → `list_staff`
- **STAFF_PROFILE_FRONTEND** Caller: `StaffRepository.listStaffWithFields(columnIds, includeInactive)` only (no `listStaff()`). `StaffListController.ensureStaffLoaded()` → `_loadStaff()` → `listStaffWithFields(columnIds, includeInactive)` with column IDs from `ColumnPreferencesService.getVisibleColumns()` or `StaffColumnConfig.getDefaultVisibleColumns()`, and `includeInactive` from `ColumnPreferencesService.getIncludeInactive()` or false. `StaffPresenter.loadStaff()` delegates to `StaffListController.loadStaffWithFields(columnIds, includeInactive)` with same preference/default logic.
- Query params: **`fields`** (required, comma-separated column IDs; 400 if missing), `include_inactive` (bool)
- Backend: **Requires `fields`**; returns 400 if missing or empty. No default columns on backend. Then `StaffService.list_all_staff(db, school_id)`; excludes admin (id=1); filters by `include_inactive`; response fields filtered by `column_ids_to_field_names(fields)` (see `sms-api/app/utils/staff_field_mapper.py`).
- Response: `{ "success": true, "data": [ { "id", "staff_id" (employee_id), "staff_name", "email", "phone", "role_id", "department_id", "status", "profile_image", ... } ], "message": "..." }`

### 1.2 Get single staff (detail view)

- **API_GET** `GET /staff/{staff_id}`
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `get_staff`
- **STAFF_PROFILE_FRONTEND** Caller: `StaffRepository.getStaff(staffId)` (e.g. from `StaffDetailView`, refresh)
- Backend: `StaffService.get_staff(db, school_id, staff_id)`; loads role permissions (PermissionRegistry for admin, else role permissions); returns full staff + `permissions` array.
- Response: `StaffDetailResponse` (full staff + permissions).

### 1.3 Create staff

- **API_POST** `POST /staff/`
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `create_staff`; **STAFF_PROFILES_MODULE** audit after create.
- **STAFF_PROFILE_FRONTEND** Caller: `StaffPresenter.createStaff(staffMember)` → `StaffRepository.createStaff(staffMember)` (from add-staff flow in `StaffListWidget` / add dialog).
- Body: `StaffCreate` (name, email, password, phone, role_id, profile_photo_url, documents, address, etc.). Backend: `StaffService.create_staff(db, school_id, staff_data)`; duplicate email → 400.

### 1.4 Update staff

- **API_PUT** `PUT /staff/{staff_id}`
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `update_staff`; `_audit_update_changes` per changed field; **STAFF_PROFILES_MODULE**.
- **STAFF_PROFILE_FRONTEND** Caller: `StaffPresenter.updateStaff(staffMember)` → `StaffRepository.updateStaff(staffMember)` (single-row save from edit form / detail).
- Body: `StaffUpdate` (partial). Backend: `StaffService.update_staff(...)`; staff id=1 protected in frontend (validation error).

### 1.5 Bulk update staff

- **API_PUT** `PUT /staff/bulk`
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `bulk_update_staff`; **STAFF_PROFILES_MODULE** audit action `bulk_update`.
- **STAFF_PROFILE_FRONTEND** Caller: `StaffPresenter.bulkUpdateStaff(staffList)` → `StaffRepository.bulkUpdateStaff(staffList)` (e.g. “Save all” in table edit mode).
- Body: `BulkStaffUpdateRequest` with `staff_list` (array of objects with `id` + update fields). Backend: `StaffService.bulk_update_staff(...)`; returns `successful` and `failed` lists; optional `fields` query for response shape.

### 1.6 Delete staff

- **API_DELETE** `DELETE /staff/{staff_id}`
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `delete_staff`; **STAFF_PROFILES_MODULE** audit; returns 204.
- **STAFF_PROFILE_FRONTEND** Caller: `StaffPresenter.deleteStaff(staffId)` → `StaffRepository.deleteStaff(staffId)` (delete from list/detail). Id=1 protected in presenter.

### 1.7 List staff by role (minimal)

- **API_GET** `GET /staff/by-role/{role_id}`
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `list_staff_by_role`
- **STAFF_PROFILE_FRONTEND** Caller: `StaffRepository.listStaffByRole(roleId)` (e.g. teacher allotment; not main Staff Profiles screen).
- Response: minimal list `[{ "id", "name" }]`.

### 1.8 Staff profile audit logs

- **API_GET** `GET /audit-logs/`
- **STAFF_PROFILE_BACKEND** Route: `sms-api/app/api/routes/audit_logs.py` → `get_audit_logs`; module name resolved from **STAFF_PROFILE_PERMISSION** submodule_id 501.
- **STAFF_PROFILE_FRONTEND** Caller: `StaffRepository.getStaffProfileAuditLogs(date: date, staffId?: staffId)` from `StaffProfilesView` when opening “View Edit History” drawer; query params: `module_id=6`, `submodule_id=501`, `date=YYYY-MM-DD`, optional `staff_id`.
- Response: `AuditLogListResponse` (data: list of `AuditLogEntry`: timestamp, action, edited_by_staff_id, description; no payload in response).

### 1.9 Profile photo upload (image used as profile_photo_url)

- **API_POST** `POST /images/upload`
- **STAFF_PROFILE_BACKEND** Route: `sms-api/app/api/routes/images.py` → `upload_image`; `ImageUploadService.upload_image` (S3 + metadata).
- **STAFF_PROFILE_FRONTEND** Caller: Staff edit and profile edit use `FileUploadService.uploadImageToServer()`, `uploadImageToServerFromBytes()`, or `WebImageService.uploadImage()` → POST `/images/upload`. The stored filename is then set as `profile_photo_url` on the staff model and sent in staff create or update API.

### 1.10 Role/department/salary minimal (when column enabled in staff module)

- **API_GET** `GET /roles/minimal` — **STAFF_PROFILE_FRONTEND** Caller: `StaffListController.loadRolesMinimal()` only when `'role'` is in visible columns (on open or on “Apply” in column settings). Data stored in `StaffListController._roles`; used for role column display and dropdown.
- **API_GET** `GET /departments/minimal` — **STAFF_PROFILE_FRONTEND** Caller: `StaffListController.loadDepartmentsMinimal()` only when `'department'` is in visible columns. Data stored in `StaffListController._departments`.
- **API_GET** salary templates (active) — **STAFF_PROFILE_FRONTEND** Caller: `StaffListController.loadSalaryTemplates()` only when `'salaryTemplate'` is in visible columns. Data stored in `StaffListController._salaryTemplates`. No GlobalService for these in the staff module.

---

## 2. STAFF_PROFILE_FRONTEND – Flutter Flow

### 2.1 Entry and screen

- **STAFF_PROFILE_VIEW** Screen: `lib/views/staff_profiles_view.dart` → `StaffProfilesView`.
- Navigation: `ModuleConfig.submoduleStaffProfiles` (501) → `navigation_view.dart` case 501 returns `StaffProfilesView()`.
- Route/slug: `staff-profiles` (from `module_config.dart`).

### 2.2 Data loading on open

1. **STAFF_PROFILE_FRONTEND** `StaffProfilesView.initState()` → `_loadDataIfNeeded()`.
2. `StaffListController.ensureStaffLoaded()` → if not already loaded: `_loadStaff()` → **STAFF_PROFILE_API** `GET /staff/?fields=...&include_inactive=...` via `StaffRepository.listStaffWithFields(columnIds, includeInactive)`. Column IDs from `ColumnPreferencesService.getVisibleColumns()` when registered, else `StaffColumnConfig.getDefaultVisibleColumns()`. Include-inactive from `ColumnPreferencesService.getIncludeInactive()` when registered, else false. Default columns are defined only in the frontend (`StaffColumnConfig.getDefaultVisibleColumns()` → e.g. profile, name, email, phone, employeeId).
3. `StaffListController.ensureColumnDataLoaded(visibleColumns)` → calls role/department/salary minimal APIs **only when those columns are enabled** in column settings: if `'role'` in visibleColumns → **API_GET** `GET /roles/minimal` via `RoleRepository.listRolesMinimal()`; if `'department'` → **API_GET** `GET /departments/minimal` via `DepartmentRepository.listDepartmentsMinimal()`; if `'salaryTemplate'` → **API_GET** salary templates (active) via `SalaryTemplateRepository.getTemplates(status: 'active')`. Data stored in `StaffListController` (_roles, _departments, _salaryTemplates); no GlobalService for this in the staff module.
4. Result: `StaffListController.staff` and `StaffListController`’s roles/departments/salaryTemplates drive the table; `StaffListWidget` gets `staff: filteredStaff`, `presenter: StaffPresenter`. Column display and dropdowns use `StaffListController` as column data provider (getRoleById, getDepartmentById, getSalaryTemplateById, roles, salaryTemplates).

### 2.3 Table and filtering

- **STAFF_PROFILE_VIEW** Body: `StaffListWidget(presenter, staff, isLoading, mode)` with `StaffTableMode.viewing` or `editing`.
- Search: `CustomSearchField` → `_filterStaff(query)` → `_searchQuery`; `_getFilteredStaff(staff, staffListController)` filters by name, email, employeeId, phone, alternatePhone, role name, department name (using `StaffListController.getRoleById`, `getDepartmentById`); excludes admin (by role name); respects `_includeInactive`.
- Include inactive: `StaffColumnSettingsDrawer` → `ColumnPreferencesService.setIncludeInactive()`; on apply, runs `StaffListController.ensureColumnDataLoaded(selected)` first (role/department/salary minimal when those columns enabled), then `StaffListController.refreshStaff()` (clear + **API_GET** list again).

### 2.4 Column preferences and list API

- **All staff list fetches** use `listStaffWithFields(columnIds, includeInactive)` only; `listStaff()` has been removed. Visible columns: `ColumnPreferencesService.getVisibleColumns()` (used in `_loadDataIfNeeded` for column data load and for `columnIds` in every list fetch). Default columns: `StaffColumnConfig.getDefaultVisibleColumns()` when preferences are not registered.
- **Role/department/salary minimal APIs** are called only when the corresponding column is enabled: on open via `ensureColumnDataLoaded(visibleColumns)`; on “Apply” in column settings via `ensureColumnDataLoaded(selected)` **before** refreshing the staff list, so the table has role/department/salary data when it re-renders.
- **STAFF_PROFILE_API** Every list request sends `fields`: `StaffRepository.listStaffWithFields(columnIds, includeInactive)` → **API_GET** `GET /staff/?fields=...&include_inactive=...`. After “Apply” in column settings: (1) `StaffListController.ensureColumnDataLoaded(selected)` (loads role/department/salary minimal when those columns are in selected), (2) then `StaffListController.refreshStaff()` → `_loadStaff()` → `listStaffWithFields(columnIds, includeInactive)` with the new visible columns from preferences.

### 2.5 Create / Edit / Delete / Bulk (presenter → repository → API)

- **STAFF_PROFILE_FRONTEND** All mutations go through `StaffPresenter` (created in `StaffProfilesView._getPresenter()`), which uses **STAFF_PROFILE_PERMISSION** (module 6, submodule 501) and **StaffListController** for list state before calling repo:
  - Create: `StaffPresenter.createStaff(staff)` → `StaffRepository.createStaff(staff)` → **API_POST** `POST /staff/`; on success `StaffListController.addStaff(created)`.
  - Update: `StaffPresenter.updateStaff(staff)` → `StaffRepository.updateStaff(staff)` → **API_PUT** `PUT /staff/{id}`; on success `StaffListController.updateStaff(updated)`.
  - Bulk: `StaffPresenter.bulkUpdateStaff(list)` → `StaffRepository.bulkUpdateStaff(list)` → **API_PUT** `PUT /staff/bulk`; on success updates `StaffListController.staff` from result.
  - Delete: `StaffPresenter.deleteStaff(id)` → `StaffRepository.deleteStaff(id)` → **API_DELETE** `DELETE /staff/{id}`; on success `StaffListController.removeStaff(id)`.
- Staff id=1 is blocked from edit/delete in presenter; bulk filters out id=1 before sending.

### 2.6 Audit log drawer

- **STAFF_PROFILE_VIEW** “View Edit History” icon → `_openAuditDrawerAndLoad()` → open drawer + `_loadAuditLogsForDate(_auditSelectedDate)`.
- **STAFF_PROFILE_FRONTEND** `StaffRepository.getStaffProfileAuditLogs(date: date)` → **API_GET** `GET /audit-logs/?module_id=6&submodule_id=501&date=YYYY-MM-DD`.
- UI: `StaffProfileAuditLogDrawer` with `StaffProfileAuditLog` model (timestamp, action, editedByStaffId, description).

### 2.7 Detail and edit screens

- Row tap (view mode): navigate to staff detail (e.g. `StaffDetailView(staffMember)`). Detail fetches full staff via **API_GET** `GET /staff/{id}` when needed. No redundant full-list refetch on return from create/edit; list updates via `StaffListController` from create/update response.
- Edit: `StaffEditView` / add flow use `StaffEditController` / same presenter; save → **API_PUT** or **API_POST** as above. Profile photo: upload via `FileUploadService.uploadImageToServer` / `uploadImageToServerFromBytes` or `WebImageService.uploadImage` (POST `/images/upload`), then set `profile_photo_url` on staff and save.

### 2.8 Key files (grep)

- View: `staff_profiles_view.dart`, `staff_detail_view.dart`, `staff_edit_view.dart`, `profile_edit_view.dart`
- Widget: `staff_list_widget.dart`, `staff_profile_audit_log_drawer.dart`, `staff_column_settings_drawer.dart`
- Controller: `staff_list_controller.dart` — staff list state and list-load API (staff, staffLoading, staffError, ensureStaffLoaded, _loadStaff, refreshStaff, setStaff, addStaff, updateStaff, removeStaff, getStaffById, loadStaffWithFields); **column data for staff table** (roles, departments, salaryTemplates; getRoleById, getDepartmentById, getSalaryTemplateById; ensureColumnDataLoaded(visibleColumnIds), loadRolesMinimal(), loadDepartmentsMinimal(), loadSalaryTemplates()). Role/department/salary minimal APIs are called only when those columns are enabled in column settings. Independent of GlobalService; staff table uses StaffListController as column data provider (getValue, getDropdownOptions, setValue in StaffColumnConfig).
- Presenter: `staff_presenter.dart`
- Repository: `staff_repository.dart`
- Model: `staff_model.dart`, `staff_profile_audit_log_model.dart`
- Config: `module_config.dart` (submoduleStaffProfiles = 501), `staff_column_config.dart` (getDefaultVisibleColumns; column data provider type `StaffColumnDataProvider` — StaffListController in staff module), `column_preferences_service.dart` (getVisibleColumns, getIncludeInactive). Export: `StaffExportService.exportToExcel(staffList, {columnDataProvider})` — when exporting from staff profiles, pass `columnDataProvider: staffListController` so role/department/salary come from StaffListController. Other modules that need staff list or lookup use `StaffListController` (e.g. SessionService, TeacherAllotmentView, SubjectsView, StudentAttendanceView, StaffEditController, StaffInlineEditController, PayrollProcessingView, StaffProfileAuditLogDrawer, CodeOfConductTab).

---

## 3. STAFF_PROFILE_BACKEND – FastAPI Flow

### 3.1 Routes and module tag

- **STAFF_PROFILE_BACKEND** All staff CRUD: `sms-api/app/api/routes/staff.py`; router prefix `"/staff"`, tag `"staff"`. Constant `STAFF_PROFILES_MODULE = "staff_profiles"` used for audit.
- Audit: `sms-api/app/api/routes/audit_logs.py` prefix `"/audit-logs"`.
- Images: `sms-api/app/api/routes/images.py` prefix `"/images"`, POST `/upload`.

### 3.2 Auth and context

- Staff routes: `Security(staff_bearer)`, `Depends(get_staff_auth_context)` → `RequestContext` (school_id, user_id, etc.). Same for audit-logs and images (staff or internal).

### 3.3 List staff (backend)

- **STAFF_PROFILE_BACKEND** `list_staff`: **Requires `fields`** query parameter; returns 400 if missing or empty (no default columns on backend). Then get all via `StaffService.list_all_staff(db, school_id)`; drop id=1; filter active by `include_inactive`; map to response using `column_ids_to_field_names(fields)` and `backend_to_response_map` (e.g. profile_photo_url → profile_image, employee_id → staff_id, name → staff_name, is_active → status). Returns JSON with `success`, `data`, `message`.

### 3.4 Get one staff (backend)

- **STAFF_PROFILE_BACKEND** `get_staff`: `StaffService.get_staff(db, school_id, staff_id)`; load role; if role_id==1 return all permission IDs else role permissions; attach `permissions` to staff dict; return `StaffDetailResponse`.

### 3.5 Create / Update / Delete (backend)

- **STAFF_PROFILE_BACKEND** `create_staff`: `StaffService.create_staff(db, school_id, StaffCreate)`; then `AuditLogger.log_audit(..., module=STAFF_PROFILES_MODULE, action="create", ...)`.
- `update_staff`: load old staff; `StaffService.update_staff(...)`; `_audit_update_changes(...)` for each changed field (audit action `"update"`, STAFF_PROFILES_MODULE).
- `delete_staff`: get staff (for name); `StaffService.delete_staff(...)`; `AuditLogger.log_audit(..., action="delete", ...)`; return 204.

### 3.6 Bulk update (backend)

- **STAFF_PROFILE_BACKEND** `bulk_update_staff`: parse `BulkStaffUpdateRequest`; build list of (id, StaffUpdate); `StaffService.bulk_update_staff(db, school_id, staff_updates)` → (successful, failed); audit with action `"bulk_update"` for successful list; return filtered or full response by `fields` query.

### 3.7 Audit logs (backend)

- **STAFF_PROFILE_BACKEND** `get_audit_logs`: required `module_id`, optional `submodule_id`, `staff_id`, `date`; resolve module name from submodule_id (501 → Staff Profiles); `AuditLogger.get_audit_logs(module=..., school_id=..., date_from/date_to, edited_by_staff_id, limit=20)`; return list of `AuditLogEntry` (no payload).

### 3.8 Services and models

- **STAFF_PROFILE_BACKEND** `StaffService`: `sms-api/app/services/staff_service.py` (create_staff, get_staff, list_all_staff, list_staff_by_role, update_staff, delete_staff, bulk_update_staff). No paginated `list_staff`; list endpoint uses `list_all_staff` only. Uses `get_dynamic_model(school_id, Staff, db)` for tenant table.
- Staff model: `sms-api/app/models/staff.py` (e.g. profile_photo_url, name, email, role_id, department_id, is_active, etc.).
- Schemas: `sms-api/app/schemas/staff.py` (StaffCreate, StaffUpdate, StaffResponse, StaffDetailResponse, BulkStaffUpdateRequest, etc.; no StaffListResponse).
- Field mapping: `sms-api/app/utils/staff_field_mapper.py` (column_ids_to_field_names; route does not use default columns).

---

## 4. STAFF_PROFILE_PERMISSION

- **STAFF_PROFILE_PERMISSION** Module: Staff Management = 6. Submodule: Staff Profiles = 501 (`ModuleConfig.submoduleStaffProfiles` / `ModuleConfig.submoduleStaffProfiles`). Slug: `staff-profiles`.
- Permission checks in Flutter: `PermissionService.canManage(6, 501)`, `canCreate(6, 501)`, `canEdit(6, 501)`, `canDelete(6, 501)` in `StaffPresenter` and `StaffProfilesView` (audit drawer, export). Backend does not re-check these in staff routes; auth is bearer + school context.

---

## 5. Summary Table (grep-friendly)

| Action | Method | Path | Frontend caller | Backend handler |
|--------|--------|------|-----------------|-----------------|
| List staff | GET | /staff/ | StaffRepository.listStaffWithFields only; StaffListController._loadStaff, StaffPresenter.loadStaff (→ loadStaffWithFields). fields required. | list_staff (fields required, 400 if missing) |
| Get staff | GET | /staff/{id} | StaffRepository.getStaff | get_staff |
| Create staff | POST | /staff/ | StaffRepository.createStaff | create_staff |
| Update staff | PUT | /staff/{id} | StaffRepository.updateStaff | update_staff |
| Bulk update | PUT | /staff/bulk | StaffRepository.bulkUpdateStaff | bulk_update_staff |
| Delete staff | DELETE | /staff/{id} | StaffRepository.deleteStaff | delete_staff |
| By role | GET | /staff/by-role/{id} | StaffRepository.listStaffByRole | list_staff_by_role |
| Audit logs | GET | /audit-logs/ | StaffRepository.getStaffProfileAuditLogs | get_audit_logs |
| Image upload | POST | /images/upload | FileUploadService.uploadImageToServer / uploadImage | upload_image |

---

## 6. Recent changes (cleanup and list API)

- **Staff module independent of GlobalService**: Staff list state and list-load API are owned by **StaffListController** (registered in `main.dart`). `StaffListController` holds `staff`, `staffLoading`, `staffError` and methods: `ensureStaffLoaded`, `refreshStaff`, `loadStaff`, `loadStaffWithFields`, `setStaff`, `addStaff`, `updateStaff`, `removeStaff`, `getStaffById`, plus document helpers. All staff-profile UI and other modules that need staff list or lookup use `StaffListController`; **GlobalService** no longer has any staff-related state or methods.
- **Role/department/salary minimal APIs (staff module only)**: Called **only when the corresponding column is enabled** in column settings. **StaffListController** owns this data and the loaders: `_roles`, `_departments`, `_salaryTemplates`; `getRoleById`, `getDepartmentById`, `getSalaryTemplateById`; `ensureColumnDataLoaded(visibleColumnIds)` (calls `loadRolesMinimal()`, `loadDepartmentsMinimal()`, `loadSalaryTemplates()` only for columns in the list); no GlobalService for column data in the staff module. On open: `_loadDataIfNeeded()` runs `ensureStaffLoaded()` and `ensureColumnDataLoaded(visibleColumns)`. On “Apply” in column settings: `ensureColumnDataLoaded(selected)` runs **first** (so role/department/salary are loaded before the table re-renders), then `refreshStaff()`. Staff table, filtering, inline edit, and export use **StaffListController** as column data provider (`StaffColumnConfig` uses `StaffColumnDataProvider`; `StaffExportService.exportToExcel(..., columnDataProvider: staffListController)` when exporting from staff profiles).
- **List staff API**: Backend requires `fields` query parameter; returns 400 if missing. No default columns on backend. Frontend uses only `listStaffWithFields(columnIds, includeInactive)`; `StaffRepository.listStaff()` removed. Default columns live in the frontend (`StaffColumnConfig.getDefaultVisibleColumns()`). `StaffListController._loadStaff()` and `StaffPresenter.loadStaff()` (→ `loadStaffWithFields`) get column IDs from `ColumnPreferencesService.getVisibleColumns()` when registered, else default; same for `includeInactive`.
- **Redundant API**: Removed full-list refetch on return from Create/Edit/Detail; list updates via `StaffListController` from create/update response. Pull-to-refresh still calls `loadStaff()` (which uses `loadStaffWithFields`).
- **Removed frontend**: `StaffPresenter.uploadProfilePhoto`; `StaffPresenter` no longer takes `FileUploadService`. Profile photo upload in staff/profile edit uses `FileUploadService.uploadImageToServer` / `uploadImageToServerFromBytes` / `WebImageService.uploadImage` only. `GlobalService.getStaffByRole` and `getStaffByDepartment` removed (unused). All references to `GlobalService` for staff (ensureStaffLoaded, refreshStaff, getStaffById, staff, addStaff, updateStaff, removeStaff) replaced with `StaffListController` in: StaffProfilesView, StaffListWidget, StaffInlineEditController, StaffColumnSettingsDrawer, SessionService, TeacherAllotmentView, SubjectsView, StudentAttendanceView, StaffEditController, PayrollProcessingView, StaffProfileAuditLogDrawer, CodeOfConductTab.
- **Removed backend**: `StaffService.list_staff` (paginated); `StaffListResponse` schema; `filter_staff_response` import from staff route. List endpoint uses `column_ids_to_field_names(fields)` only (no default column fallback).

---

Use the tags **STAFF_PROFILE_API**, **STAFF_PROFILE_FRONTEND**, **STAFF_PROFILE_BACKEND**, **STAFF_PROFILE_PERMISSION**, **STAFF_PROFILE_VIEW**, and **API_GET** / **API_POST** / **API_PUT** / **API_DELETE** to grep this document quickly.
