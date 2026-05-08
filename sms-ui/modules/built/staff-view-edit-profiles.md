---
title: Staff View and Edit Staff Profiles
type: module
project: sms-ui
module: Staff Management
last-updated: 2026-03-18
---

# Staff View and Edit Staff Profiles – Complete Documentation

STAFF_VIEW_EDIT_DOCUMENT: Master reference for staff view, staff edit, profile view, and profile edit flows. Grep-friendly tags for quick lookup.

---

## GREP QUICK REFERENCE

| Grep for | Meaning |
|----------|---------|
| `STAFF_PROFILES_VIEW` | Main staff list screen (StaffProfilesView) |
| `STAFF_DETAIL_VIEW` | Individual staff detail screen (StaffDetailView) |
| `STAFF_EDIT_VIEW` | Staff create/edit form (StaffEditView) |
| `PROFILE_VIEW` | Logged-in user's profile (ProfileView) |
| `PROFILE_EDIT_VIEW` | Logged-in user edits own profile (ProfileEditView) |
| `STAFF_PROFILE_API` | API endpoints used |
| `STAFF_PROFILE_FRONTEND` | Flutter flow, callers, widgets |
| `STAFF_PROFILE_BACKEND` | FastAPI routes and services |
| `STAFF_PROFILE_PERMISSION` | Permission/module IDs |
| `NAVIGATION` | How views are reached |
| `STAFF_MODEL` | Staff data structure |
| `API_GET` / `API_POST` / `API_PUT` / `API_DELETE` | HTTP method |

---

## 1. NAVIGATION – How Views Are Reached

### 1.1 STAFF_PROFILES_VIEW

- **NAVIGATION** Entry: `ModuleConfig.submoduleStaffProfiles` (501) → `navigation_view.dart` case 501 returns `StaffProfilesView()`.
- **NAVIGATION** Route/slug: `staff-profiles` (from `module_config.dart`).
- **STAFF_PROFILES_VIEW** File: `lib/views/staff_profiles_view.dart`.

### 1.2 STAFF_DETAIL_VIEW

- **NAVIGATION** Reached from: Row tap in `StaffListWidget` (view mode) → `Navigator.push(MaterialPageRoute(builder: (context) => StaffDetailView(staffMember: staffMember)))`.
- **STAFF_DETAIL_VIEW** File: `lib/views/staff_detail_view.dart`.
- **NAVIGATION** Parent: StaffProfilesView → StaffListWidget → row tap.

### 1.3 STAFF_EDIT_VIEW

- **NAVIGATION** Create: Add button in StaffProfilesView opens end drawer → `StaffColumnSettingsDrawer` (or add flow in StaffListWidget) → `Navigator.push(StaffEditView())` (null staff = create).
- **NAVIGATION** Edit: StaffDetailView "Edit" button → `Navigator.push(StaffEditView(staffMember: _currentStaff))`.
- **STAFF_EDIT_VIEW** File: `lib/views/staff_edit_view.dart`.
- **NAVIGATION** Note: Add button in AppBar opens column settings (end drawer); add flow is in StaffListWidget.

### 1.4 PROFILE_VIEW

- **NAVIGATION** Entry: Sidebar/profile menu or route `/profile` → `ProfileView()`.
- **NAVIGATION** File: `lib/views/profile_view.dart`.
- **NAVIGATION** Shows logged-in user's own profile from `SessionService.currentSession?.staff`.

### 1.5 PROFILE_EDIT_VIEW

- **NAVIGATION** Reached from: ProfileView "Edit" button → `Navigator.push(ProfileEditView(staffMember: staff))`.
- **PROFILE_EDIT_VIEW** File: `lib/views/profile_edit_view.dart`.
- **NAVIGATION** Edit button hidden for admin users (role name contains 'admin').

---

## 2. STAFF_PROFILES_VIEW – Main Staff List

### 2.1 Structure

- **STAFF_PROFILES_VIEW** Widget: `StaffProfilesView` (StatefulWidget).
- **STAFF_PROFILES_VIEW** Body: `StaffListWidget(presenter, staff, isLoading, mode)`.
- **STAFF_PROFILES_VIEW** Modes: `StaffTableMode.viewing` | `StaffTableMode.editing` (toggle in AppBar).
- **STAFF_PROFILES_VIEW** Search: `CustomSearchField` → `_filterStaff(query)` filters by name, email, employeeId, phone, alternatePhone, role name, department name.
- **STAFF_PROFILES_VIEW** Excludes: Admin staff (role name contains 'admin'); inactive staff unless `_includeInactive` is true.
- **STAFF_PROFILES_VIEW** Column data: `StaffListController` (roles, departments, salaryTemplates) via `ColumnPreferencesService.getVisibleColumns()`.

### 2.2 Data Loading

- **STAFF_PROFILE_FRONTEND** `initState()` → `_loadDataIfNeeded()`.
- **STAFF_PROFILE_FRONTEND** `StaffListController.ensureStaffLoaded()` → `listStaffWithFields(columnIds, includeInactive)`.
- **STAFF_PROFILE_FRONTEND** `StaffListController.ensureColumnDataLoaded(visibleColumns)` → role/department/salary minimal APIs when those columns are enabled.

### 2.3 Actions (AppBar)

- **STAFF_PROFILES_VIEW** Add Column: Opens `StaffColumnSettingsDrawer` (end drawer).
- **STAFF_PROFILES_VIEW** View Edit History: Opens `StaffProfileAuditLogDrawer` (drawer); requires `canManage(6, 501)`.
- **STAFF_PROFILES_VIEW** Export to Excel: `StaffExportService.exportToExcel()`; requires `canManage(6, 501)`.
- **STAFF_PROFILES_VIEW** Mode toggle: View / Edit (affects table editability).

### 2.4 Row Tap Behaviour

- **STAFF_PROFILE_FRONTEND** View mode: Row tap → `StaffDetailView(staffMember)`.
- **STAFF_PROFILE_FRONTEND** Edit mode: Inline editing via `StaffInlineEditController` (no navigation to detail).

### 2.5 Dependencies

- **STAFF_PROFILES_VIEW** Controllers: `StaffListController`, `ColumnPreferencesService`, `StaffExportService`, `PermissionService`.
- **STAFF_PROFILES_VIEW** Presenter: `StaffPresenter` (for CRUD).
- **STAFF_PROFILES_VIEW** Widgets: `StaffListWidget`, `CustomSearchField`, `StaffProfileAuditLogDrawer`, `StaffColumnSettingsDrawer`.

---

## 3. STAFF_DETAIL_VIEW – Individual Staff Detail

### 3.1 Structure

- **STAFF_DETAIL_VIEW** Widget: `StaffDetailView` (StatefulWidget) with `SingleTickerProviderStateMixin`.
- **STAFF_DETAIL_VIEW** Input: `Staff staffMember` (from list row).
- **STAFF_DETAIL_VIEW** Tabs: 7 tabs (Personal, Classes, Timetable, Attendance, Payroll, Documents, History).
- **STAFF_DETAIL_VIEW** Data: Fetches full staff via `StaffRepository.getStaff(id, refresh: true)` on init; uses `_fullStaffDetails ?? widget.staffMember` as `_currentStaff`.

### 3.2 Tab Content

- **STAFF_DETAIL_VIEW** Personal: Date of birth, gender, blood group; contact (email, phone, alternate); address; emergency contact; employment (qualification, experience).
- **STAFF_DETAIL_VIEW** Classes: Assigned sections from `_currentStaff.assignedSectionIds` + `GlobalService.sections`, grouped by class.
- **STAFF_DETAIL_VIEW** Timetable: Placeholder ("Timetable information will be displayed here").
- **STAFF_DETAIL_VIEW** Attendance: `_MonthlyAttendanceTab` – calendar view, `StaffAttendancePresenter.loadStaffMonthlyAttendance()`.
- **STAFF_DETAIL_VIEW** Payroll: Placeholder.
- **STAFF_DETAIL_VIEW** Documents: List of `_currentStaff.documents`.
- **STAFF_DETAIL_VIEW** History: Created at, last updated, account status, login access, last login, joined date.

### 3.3 Edit Button

- **STAFF_DETAIL_VIEW** Shown only if `PermissionService.canEdit(6, 501)`.
- **STAFF_DETAIL_VIEW** Action: `_navigateToEdit()` → `StaffEditView(staffMember: _currentStaff)`; on save, `_loadFullStaffDetails()` then `Navigator.pop(true)`.

### 3.4 AppBar

- **STAFF_DETAIL_VIEW** Expanded AppBar: Profile photo, name, email, employee ID; chips for role, department, phone, joined date, status.
- **STAFF_DETAIL_VIEW** Role/Department: From `GlobalService.roles`, `GlobalService.departments` (Obx).

### 3.5 Profile Photo

- **STAFF_DETAIL_VIEW** `ImageUrlService.getImageUrl(schoolId, _currentStaff.profilePhotoUrl)`; fallback gradient + person icon if no photo.

---

## 4. STAFF_EDIT_VIEW – Create/Edit Staff

### 4.1 Structure

- **STAFF_EDIT_VIEW** Widget: `StaffEditView` (StatefulWidget); `staffMember` null = create, non-null = edit.
- **STAFF_EDIT_VIEW** Controller: `StaffEditController` (GetX, tag: `staff_edit_${id}` or `staff_edit_new`).
- **STAFF_EDIT_VIEW** Layout: 6-step wizard (Basic Info, Personal, Employment, Payroll, Documents, Access).
- **STAFF_EDIT_VIEW** PopScope: Prevents back without discard confirmation if `hasUnsavedChanges`.
- **STAFF_EDIT_VIEW** Navigation back: `_navigateBack()` pops edit view, then detail view (if came from detail).

### 4.2 Steps and Fields

- **STAFF_EDIT_VIEW** Step 1 – Basic Info: Profile photo (`ProfilePhotoWidget`), name, email, phone, alternate phone.
- **STAFF_EDIT_VIEW** Step 2 – Personal: Date of birth, gender, blood group; address (`AddressFormWidget`); emergency contact (name, phone, relation).
- **STAFF_EDIT_VIEW** Step 3 – Employment: Role (dropdown from `GlobalService.roles`), department (read-only from role), employee ID, joining date, qualification, experience; assigned sections (`SectionSelectionDrawer`).
- **STAFF_EDIT_VIEW** Step 4 – Payroll: Salary template (dropdown from `SalaryTemplatePresenter`), monthly gross salary, bank account, IFSC, bank name, branch.
- **STAFF_EDIT_VIEW** Step 5 – Documents: `DocumentUploadWidget` (add/remove documents).
- **STAFF_EDIT_VIEW** Step 6 – Access: Login allowed (AppSwitch), password (required when enabling login; optional when editing existing login-allowed staff).
- **STAFF_EDIT_VIEW** Deactivate (edit only): "Danger Zone" with "Deactivate Staff Member" button; calls `StaffPresenter.deactivateStaff()`.

### 4.3 Validation

- **STAFF_EDIT_VIEW** Required: name, email, phone (10 digits), role, employee ID.
- **STAFF_EDIT_VIEW** Optional: alternate phone (10 digits if provided), date of birth (DD/MM/YYYY), joining date.
- **STAFF_EDIT_VIEW** Protected staff (id=1): Cannot edit; shows warning banner; save/deactivate disabled.

### 4.4 Save Flow

- **STAFF_PROFILE_FRONTEND** Create: `StaffEditController.saveStaff()` → `StaffPresenter.createStaff()` → `StaffRepository.createStaff()` → `API_POST /staff/`.
- **STAFF_PROFILE_FRONTEND** Edit: `StaffEditController.saveStaff()` → `StaffPresenter.updateStaff()` → `StaffRepository.updateStaff()` → `API_PUT /staff/{id}`.
- **STAFF_PROFILE_FRONTEND** Profile photo: `FileUploadService.uploadImageToServer` / `uploadImageToServerFromBytes` or `WebImageService.uploadImage` → POST `/images/upload`; filename set as `profile_photo_url` on staff.

---

## 5. PROFILE_VIEW – Logged-in User's Profile

### 5.1 Structure

- **PROFILE_VIEW** Widget: `ProfileView` (StatefulWidget) with `SingleTickerProviderStateMixin`.
- **PROFILE_VIEW** Data source: `SessionService.currentSession?.staff`, `SessionService.currentSession?.role`, `SessionService.currentSession?.department`.
- **PROFILE_VIEW** Tabs: 7 tabs (Personal, Classes, Timetable, Attendance, Payroll, Documents, History).

### 5.2 Tab Content

- **PROFILE_VIEW** Personal: Name, email, phone, alternate phone, role, department; address (if present); Settings (Dark Mode toggle via `ThemeController`).
- **PROFILE_VIEW** Classes: Assigned sections from session staff; same layout as StaffDetailView.
- **PROFILE_VIEW** Timetable: Placeholder.
- **PROFILE_VIEW** Attendance: `_ProfileMonthlyAttendanceTab` (same calendar as StaffDetailView).
- **PROFILE_VIEW** Payroll: Placeholder.
- **PROFILE_VIEW** Documents: List with download icon (TODO: implement download).
- **PROFILE_VIEW** History: Created at, last updated, last login; Activity Log placeholder.

### 5.3 Edit Button

- **PROFILE_VIEW** Hidden for admin users.
- **PROFILE_VIEW** Action: `_navigateToEdit()` → `ProfileEditView(staffMember: staff)`; on save, `sessionService.initializeSession(staffId)` to refresh session.

### 5.4 Logout

- **PROFILE_VIEW** AppBar: Logout button → `AppDialog.showConfirmDialog` → `AuthRepository.logout()` → `SessionService.clearSession()` → `context.go(loginRoute)`.

---

## 6. PROFILE_EDIT_VIEW – Edit Own Profile

### 6.1 Structure

- **PROFILE_EDIT_VIEW** Widget: `ProfileEditView` (StatelessWidget).
- **PROFILE_EDIT_VIEW** Input: `Staff staffMember` (from session).
- **PROFILE_EDIT_VIEW** Controller: `ProfileEditController` (GetX, tag: `profile_edit_${staffMember.id}`).
- **PROFILE_EDIT_VIEW** Layout: Single scrollable form (no steps).

### 6.2 Editable vs Read-only

- **PROFILE_EDIT_VIEW** Editable: Name, alternate phone, date of birth, gender, blood group; address; emergency contact; qualification; experience; profile photo.
- **PROFILE_EDIT_VIEW** Read-only: Email, phone (managed by school).
- **PROFILE_EDIT_VIEW** Documents: Read-only list; "Documents are managed by the school".

### 6.3 Save Flow

- **PROFILE_EDIT_VIEW** `ProfileEditController.saveProfile()` → `StaffPresenter.updateStaff()` → `StaffRepository.updateStaff()` → `API_PUT /staff/{id}`.
- **PROFILE_EDIT_VIEW** Profile photo: Same upload flow as StaffEditView.
- **PROFILE_EDIT_VIEW** On success: `Get.delete<ProfileEditController>(tag)`; `Navigator.pop(true)`; ProfileView refreshes session.

### 6.4 Sections

- **PROFILE_EDIT_VIEW** Profile Photo, Basic Information, Additional Personal Information, Address, Emergency Contact, Qualification & Experience, Documents (read-only), Save Changes button.

---

## 7. STAFF_MODEL – Data Structure

### 7.1 Core Fields

- **STAFF_MODEL** id, name, email, phone, alternatePhone, roleId, assignedSectionIds.
- **STAFF_MODEL** profilePhotoUrl, dateOfBirth, gender, bloodGroup.
- **STAFF_MODEL** emergencyContactName, emergencyContactPhone, emergencyContactRelation.
- **STAFF_MODEL** address (Address?), documents (List<StaffDocument>).
- **STAFF_MODEL** joiningDate, employeeId, qualification, experience.
- **STAFF_MODEL** salaryTemplateId, monthlyGrossSalary, bankAccountNumber, bankIfscCode, bankName, bankBranchName.
- **STAFF_MODEL** departmentId, isActive, lastLoginAt, isLoginAllowed, password (write-only), permissions.
- **STAFF_MODEL** createdAt, updatedAt.

### 7.2 API Parsing

- **STAFF_MODEL** `Staff.fromApiJson()`: Handles list format (staff_id, staff_name, status, profile_image) and detail format (full snake_case).
- **STAFF_MODEL** `Staff.fromMinimalApiJson()`: id, name only (for by-role).

---

## 8. STAFF_PROFILE_API – Endpoints

### 8.1 List Staff

- **API_GET** `GET /staff/`
- **STAFF_PROFILE_API** Query: `fields` (required), `include_inactive`.
- **STAFF_PROFILE_FRONTEND** Caller: `StaffRepository.listStaffWithFields()`.
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `list_staff`.

### 8.2 Get Staff

- **API_GET** `GET /staff/{id}`
- **STAFF_PROFILE_FRONTEND** Caller: `StaffRepository.getStaff()`.
- **STAFF_PROFILE_FRONTEND** Used by: StaffDetailView, ProfileView (via session refresh), ProfileEditController.
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `get_staff`.

### 8.3 Create Staff

- **API_POST** `POST /staff/`
- **STAFF_PROFILE_FRONTEND** Caller: `StaffRepository.createStaff()`.
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `create_staff`.

### 8.4 Update Staff

- **API_PUT** `PUT /staff/{id}`
- **STAFF_PROFILE_FRONTEND** Caller: `StaffRepository.updateStaff()`.
- **STAFF_PROFILE_FRONTEND** Used by: StaffEditView, ProfileEditView.
- **STAFF_PROFILE_BACKEND** Route: `staff.py` → `update_staff`.

### 8.5 Image Upload

- **API_POST** `POST /images/upload`
- **STAFF_PROFILE_FRONTEND** Caller: `FileUploadService.uploadImageToServer`, `uploadImageToServerFromBytes`, `WebImageService.uploadImage`.
- **STAFF_PROFILE_FRONTEND** Used by: StaffEditController, ProfileEditController for profile photo.

### 8.6 Audit Logs

- **API_GET** `GET /audit-logs/?module_id=6&submodule_id=501&date=YYYY-MM-DD`
- **STAFF_PROFILE_FRONTEND** Caller: `StaffRepository.getStaffProfileAuditLogs()`.
- **STAFF_PROFILE_FRONTEND** Used by: StaffProfileAuditLogDrawer.

---

## 9. STAFF_PROFILE_PERMISSION

- **STAFF_PROFILE_PERMISSION** Module: Staff Management = 6.
- **STAFF_PROFILE_PERMISSION** Submodule: Staff Profiles = 501.
- **STAFF_PROFILE_PERMISSION** Checks: `canManage(6, 501)` (audit, export), `canEdit(6, 501)` (detail edit button).
- **STAFF_PROFILE_PERMISSION** Profile edit: No permission check; admin users cannot see edit button (role-based).

---

## 10. Key Files (Grep-Friendly)

| Component | File |
|-----------|------|
| STAFF_PROFILES_VIEW | lib/views/staff_profiles_view.dart |
| STAFF_DETAIL_VIEW | lib/views/staff_detail_view.dart |
| STAFF_EDIT_VIEW | lib/views/staff_edit_view.dart |
| PROFILE_VIEW | lib/views/profile_view.dart |
| PROFILE_EDIT_VIEW | lib/views/profile_edit_view.dart |
| StaffEditController | lib/controllers/staff_edit_controller.dart |
| ProfileEditController | lib/controllers/profile_edit_controller.dart |
| StaffListController | lib/controllers/staff_list_controller.dart |
| StaffPresenter | lib/presenters/staff_presenter.dart |
| StaffRepository | lib/repositories/staff_repository.dart |
| STAFF_MODEL | lib/models/staff_model.dart |
| StaffListWidget | lib/widgets/staff_list_widget.dart |
| StaffProfileAuditLogDrawer | lib/widgets/staff_profile_audit_log_drawer.dart |
| StaffColumnSettingsDrawer | lib/widgets/staff_column_settings_drawer.dart |
| ProfilePhotoWidget | lib/widgets/profile_photo_widget.dart |
| AddressFormWidget | lib/widgets/address_form_widget.dart |
| DocumentUploadWidget | lib/widgets/document_upload_widget.dart |
| SectionSelectionDrawer | lib/widgets/section_selection_drawer.dart |
| ModuleConfig | lib/config/module_config.dart |

---

## 11. Differences: Staff Edit vs Profile Edit

| Aspect | StaffEditView | ProfileEditView |
|--------|---------------|-----------------|
| STAFF_EDIT_VIEW vs PROFILE_EDIT_VIEW | Full staff create/edit | Own profile edit only |
| Steps | 6-step wizard | Single scroll form |
| Role/Department | Editable (role dropdown) | Read-only (from session) |
| Employee ID | Editable | Not shown |
| Email/Phone | Editable (create) / full (edit) | Read-only |
| Payroll | Editable | Not shown |
| Login/Password | Editable | Not shown |
| Documents | Add/remove | Read-only list |
| Deactivate | Yes (edit mode) | No |
| Controller | StaffEditController | ProfileEditController |
| Permission | canEdit(6, 501) for edit button | None (admin cannot edit) |

---

## 12. Flow Summary

```
STAFF_PROFILES_VIEW
  ├── Row tap (view) → STAFF_DETAIL_VIEW
  │     └── Edit → STAFF_EDIT_VIEW (edit)
  ├── Add (from StaffListWidget) → STAFF_EDIT_VIEW (create)
  └── Column settings / Audit / Export

Profile menu → PROFILE_VIEW
  └── Edit (non-admin) → PROFILE_EDIT_VIEW
```

---

Use the tags **STAFF_PROFILES_VIEW**, **STAFF_DETAIL_VIEW**, **STAFF_EDIT_VIEW**, **PROFILE_VIEW**, **PROFILE_EDIT_VIEW**, **STAFF_PROFILE_API**, **STAFF_PROFILE_FRONTEND**, **STAFF_PROFILE_BACKEND**, **STAFF_PROFILE_PERMISSION**, **NAVIGATION**, **STAFF_MODEL**, **API_GET**, **API_POST**, **API_PUT** to grep this document quickly.
