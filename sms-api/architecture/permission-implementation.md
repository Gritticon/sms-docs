---
title: Backend Permission Implementation Guide
type: architecture
project: sms-api
last-updated: 2026-03-18
---

# Backend Permission Implementation Guide

This document provides detailed specifications for implementing the permission system in the backend using **Python with SQLAlchemy** to work seamlessly with the Flutter frontend.

## Table of Contents
1. [Overview](#overview)
2. [Permission Model Philosophy](#permission-model-philosophy)
3. [Permission Structure](#permission-structure)
4. [Permission ID Assignment](#permission-id-assignment)
5. [Database Schema Design](#database-schema-design)
6. [Role Model Implementation](#role-model-implementation)
7. [API Response Format](#api-response-format)
8. [Permission Validation](#permission-validation)
9. [Permission Logic](#permission-logic)
10. [Examples](#examples)
11. [Best Practices](#best-practices)

---

## Overview

The frontend expects:
- **No separate permissions API** - permissions are embedded in the role API response
- **Integer permission IDs** (not strings) - sequential IDs from 1 to 180
- **Permission IDs in role response** - role API includes a `permissions` array with integer IDs
- **Fixed ID mapping** - permission IDs must match the frontend's PermissionRegistry order
- **Module-based read access** - If a module exists, read access is implicit
- **Explicit write permissions** - Only create, edit, delete, and manage actions require explicit permissions

---

## Permission Model Philosophy

### Core Concept

**Module Access = Read Access (Implicit)**
- If a user's role has access to a module, they automatically have **read/view** permissions for that module
- No explicit "view" permission ID is needed for basic read operations
- Module access is determined by the presence of **any permission** for that module/submodule

**Explicit Permissions = Write Access**
- Only **create**, **edit**, **delete**, and **manage** actions require explicit permission IDs
- These permission IDs must be stored in the role's permissions array
- Without explicit permission IDs, users can only read/view data

### Permission Types

1. **Implicit Read Permissions** (No ID needed)
   - Automatically granted when module/submodule has any permission
   - Allows: Viewing data, reading records, listing items

2. **Explicit Write Permissions** (Requires permission IDs)
   - **Create** (ID: base + 1): Create new records
   - **Edit** (ID: base + 2): Modify existing records
   - **Delete** (ID: base + 3): Remove records
   - **Manage** (ID: base + 4): Full administrative control and advanced operations

### Understanding "Manage" vs "Read" Operations

**Read Operations (Implicit - No Permission ID Required):**
- **View/List**: Displaying data, viewing records, listing items
- **Read Details**: Viewing individual record details
- **Search/Filter**: Searching and filtering data
- **Export/Print**: Exporting or printing data (read-only operations)
- **Dashboard Access**: Viewing dashboards, reports, and analytics

**Manage Operations (Explicit Permission ID Required):**
- **Administrative Functions**: System configuration, settings management
- **Bulk Operations**: Bulk updates, bulk deletions, bulk imports
- **Permission Management**: Assigning permissions to other users/roles
- **Advanced Configuration**: Module-specific advanced settings
- **Override Operations**: Overriding restrictions or validations
- **System-Level Changes**: Making changes that affect system behavior
- **Approval Workflows**: Approving/rejecting requests, workflows
- **Audit Management**: Managing audit logs, system logs
- **Data Management**: Importing/exporting data, data migration
- **Role Assignment**: Assigning roles to users

**Key Differences:**

| Aspect | Read (Implicit) | Manage (Explicit) |
|--------|-----------------|-------------------|
| **Purpose** | Viewing and consuming data | Controlling and configuring system |
| **Scope** | Individual records or lists | System-wide or bulk operations |
| **Impact** | No data modification | Can modify system behavior |
| **Complexity** | Simple data retrieval | Administrative and advanced operations |
| **Examples** | View student list, see student details | Assign roles, configure module settings, bulk import students |
| **Permission Required** | None (implicit with module access) | Explicit "manage" permission ID |

**Practical Examples:**

**Student Management Module:**

- **Read (No Permission Needed)**: 
  - View list of students
  - View student profile details
  - Search students by name/ID
  - Export student list to CSV
  - View student attendance history

- **Create (Permission ID Required)**:
  - Add new student record
  - Enroll new student

- **Edit (Permission ID Required)**:
  - Update student information
  - Modify student profile

- **Delete (Permission ID Required)**:
  - Remove student record
  - Delete student data

- **Manage (Permission ID Required)**:
  - Bulk import students from CSV
  - Assign students to classes in bulk
  - Configure student module settings
  - Manage student enrollment workflows
  - Approve/reject student applications
  - Configure student data validation rules

**Staff Management Module:**

- **Read (No Permission Needed)**:
  - View staff list
  - View staff profile
  - View staff attendance records

- **Create (Permission ID Required)**:
  - Add new staff member

- **Edit (Permission ID Required)**:
  - Update staff information

- **Delete (Permission ID Required)**:
  - Remove staff member

- **Manage (Permission ID Required)**:
  - Assign roles to staff members
  - Configure department settings
  - Manage payroll settings
  - Approve staff leave requests
  - Configure staff module permissions
  - Bulk update staff records

### Example Scenarios

**Scenario 1: Read-Only Access**
- Role has access to "Student Management" module
- No explicit permissions assigned
- **Result**: User can view/list students but cannot create, edit, or delete

**Scenario 2: Full Access**
- Role has access to "Student Management" module
- Explicit permissions: [create, edit, delete, manage] IDs assigned
- **Result**: User can view, create, edit, delete, and manage students

**Scenario 3: Limited Write Access**
- Role has access to "Student Management" module
- Explicit permissions: [create, edit] IDs assigned (no delete/manage)
- **Result**: User can view, create, and edit students but cannot delete or manage

---

## Permission Structure

### Permission ID Reference

Each permission represents a specific **write action** on a module/submodule. The permission ID encodes:
- **Module ID** (1–12): Which module the permission applies to
- **Submodule ID**: Which submodule within the module (or same as module_id if no submodules)
- **Action Type**: One of "create", "edit", "delete", "manage"

**Note**: "view" action IDs exist in the system but are **not required** for read access. They are included for consistency but can be ignored if module access is granted.

### Permission Fields (Reference Only)

| Field | Type | Description |
|-------|------|-------------|
| `id` | Integer | Unique permission ID (1-180) |
| `module_id` | Integer | Module ID (1-12) |
| `submodule_id` | Integer | Submodule ID (or same as module_id) |
| `action` | String | One of: "create", "edit", "delete", "manage" (view is implicit) |

**Important**: Only the **permission ID** (integer) is stored in the database. The module_id, submodule_id, and action are derived from the ID using the PermissionRegistry mapping.

**View permission for read endpoints:** Some read endpoints (e.g. GET for a submodule) use `check_staff_permission(..., action="view")`. For those, the role must have the **view** permission ID for that submodule. Full-access roles (e.g. Admin) should have all 1–180 IDs (view + create + edit + delete + manage). The script `sms-api/scripts/update_admin_permissions.py` uses `PermissionRegistry.get_all_permission_ids()` and should be re-run after any change to `app/core/permissions.py`.

---

## Permission ID Assignment

**CRITICAL**: Permission IDs must be assigned in the exact order shown below. The frontend's `PermissionRegistry` generates permissions in this order, and IDs must match.

### ID Assignment Rules

1. **Sequential Ordering**: IDs are assigned sequentially from 1 to 180
2. **Module Order**: Modules are processed in order (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 12)
3. **Submodule Order**: Within each module, submodules are processed in sorted order
4. **Action Order**: For each submodule, actions are assigned in order: view, create, edit, delete, manage

### Complete ID Mapping

| ID Range | Module | Submodule | Actions (view, create, edit, delete, manage) |
|----------|--------|-----------|-----------------------------------------------|
| 1-5 | 1 (Dashboard) | 1 | Dashboard permissions |
| 6-20 | 2 (Communication Hub) | 101 (Messages), 102 (Notifications), 103 (Announcements) | Per submodule |
| 21-35 | 3 (Transport) | 201 (Routes), 202 (Vehicles), 203 (Drivers) | Per submodule |
| 36-70 | 4 (Student) | 301 (Student Profiles), 305 (House Tag), 306 (Certificates), 307 (Documents), 308 (Achievements), 309 (Code of Conduct), 311 (Requests) | Per submodule |
| 71-90 | 5 (Exam) | 400 (Exams), 402 (Question Papers), 403 (Results), 404 (Report Cards). 401 (Exam Schedule) reserved – combined under 400 | Per submodule |
| 91-110 | 6 (Staff) | 501 (Staff Profiles), 504 (Staff Attendance), 505 (Payroll), 507 (Roles & Departments) | Per submodule |
| 111-130 | 7 (Class) | 601 (Classes & Sections), 605 (Subjects), 603 (Assignments), 604 (Class Attendance) | Per submodule |
| 131-140 | 8 (Complaints) | 701 (View Complaints), 702 (Resolve Complaints) | Per submodule |
| 141-150 | 9 (School Management) | 801 (School Info), 802 (School Settings) | Per submodule |
| 151-170 | 10 (Timetable) | 602 (Timetable), 606 (Teacher Allotment), 607 (Holidays), 608 (Substitute) | Per submodule |
| 171-180 | 12 (Class Session) | 901 (Sessions), 902 (Class Progress) | Per submodule |

### Action Order (for each submodule)

For each submodule, actions are assigned in this order:
1. `view` (ID: base) - **Not required for read access** (implicit)
2. `create` (ID: base + 1) - **Required for creating records**
3. `edit` (ID: base + 2) - **Required for editing records**
4. `delete` (ID: base + 3) - **Required for deleting records**
5. `manage` (ID: base + 4) - **Required for full management**

**Example**: For Messages submodule (starts at ID 6):
- ID 6: view (implicit, not needed for legacy logic; **required** for endpoints that call `check_staff_permission(..., action="view")`)
- ID 7: create (explicit permission needed)
- ID 8: edit (explicit permission needed)
- ID 9: delete (explicit permission needed)
- ID 10: manage (explicit permission needed)

### Frontend/backend permission ID alignment

The **permission matrix** in the Flutter app (role edit screen) and **role save** use permission IDs produced by `PermissionRegistry.getAllPermissions()` in `sms/lib/services/permission_registry.dart`, which iterates `ModuleConfig.moduleSubmodules` in definition order. The **backend** assigns IDs in `app/core/permissions.py` using **sorted module IDs** and **sorted submodule IDs** per module.

For the same (module_id, submodule_id, action) to have the **same numeric ID** on both sides:

- **Flutter** `ModuleConfig.moduleDefinitions` (and thus `moduleSubmodules`) should use the **same module order** as the backend: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 12 (no module 11 in backend; module 12 is Class Session).
- **Submodule order** within each module should match the backend’s sorted order (e.g. Staff: 501, 504, 505, 507; Class: 601, 603, 604, 605).

If the orders differ, the role edit matrix will show or save the wrong permissions, and API permission checks may fail. See `fix/PERMISSIONS-NEW-MODULES-SUBMODULES.md` for the checklist when adding submodules.

---

## Database Schema Design

### Roles Table Structure

The `roles` table should include a JSON column to store permission IDs as an array of integers.

**Required Columns:**
- `id`: Integer primary key (auto-increment)
- `name`: String (unique, required)
- `description`: Text (optional)
- `department_id`: Integer foreign key (optional, references departments table)
- `permissions`: JSON column storing array of integer permission IDs
- `created_at`: DateTime timestamp
- `updated_at`: DateTime timestamp

**Key Design Decisions:**
1. **JSON Column Type**: Use SQLAlchemy's `JSON` type for the permissions column
2. **Default Value**: Set default to empty array `[]` (not null)
3. **No Separate Permissions Table**: Permissions are stored directly in the role model
4. **No Junction Table**: No need for role_permissions relationship table

### Database Compatibility

- **PostgreSQL**: Use `JSON` or `JSONB` column type (JSONB recommended for better performance)
- **MySQL 5.7+**: Use `JSON` column type (native JSON support)
- **MySQL 5.6 or older**: Use `TEXT` column type and handle JSON serialization in application code
- **SQLite**: Use `JSON` column type (SQLAlchemy handles serialization automatically)
- **SQL Server**: Use `NVARCHAR(MAX)` or `TEXT` column type with JSON serialization

SQLAlchemy's `JSON()` type automatically handles serialization/deserialization for most modern databases.

### Indexes

Create an index on `department_id` for faster role lookups when filtering by department.

---

## Role Model Implementation

### Model Structure

The Role model should include:

1. **Basic Fields**: id, name, description, department_id, timestamps
2. **Permissions Field**: JSON column storing array of integer IDs
3. **Helper Methods**: Methods for managing and validating permissions

### Required Methods

**Permission Management:**
- `set_permissions(permission_ids: list[int])`: Set permissions with validation
- `add_permission(permission_id: int)`: Add a single permission
- `remove_permission(permission_id: int)`: Remove a single permission
- `has_permission(permission_id: int) -> bool`: Check if role has specific permission
- `get_permissions() -> list[int]`: Get list of permission IDs

**Validation:**
- Validate that all permission IDs are integers
- Validate that all permission IDs are in range (1-180)
- Remove duplicates automatically
- Sort permission IDs for consistency

**Serialization:**
- `to_dict()`: Convert role to dictionary for API response
- Ensure permissions field is always returned as array (never null)

### Permission Registry Helper

Create a `PermissionRegistry` helper class (not a database model) that:

1. **Defines Module Configuration**: Maps module IDs to their submodules
2. **Validates Permission IDs**: Checks if IDs are valid (1-180)
3. **Maps IDs to Details**: Converts permission ID to (module_id, submodule_id, action)
4. **Provides Reference Data**: Returns all valid permission IDs, module lists, etc.

This helper is used for validation and reference only - it does not interact with the database.

---

## API Response Format

### GET /roles/{role_id}

**Response Structure:**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Admin",
    "description": "Administrator role with full access",
    "department_id": 1,
    "permissions": [2, 3, 4, 5, 7, 8, 9, 10], 
    "created_at": "2025-12-02T02:39:08",
    "updated_at": "2025-12-02T02:39:08"
  },
  "message": "Operation successful"
}
```

**Key Points:**
- `permissions` field may contain **all permission IDs** (1–180), including view, so that read endpoints that check `action="view"` work for the role.
- For full-access roles (e.g. Admin), store all 1–180 (use `PermissionRegistry.get_all_permission_ids()`; the script `update_admin_permissions.py` does this).
- Return empty array `[]` if role has no permissions.
- All IDs must be integers (not strings).
- IDs should be sorted for consistency.

### GET /roles/

**List Response Structure:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Admin",
      "permissions": [2, 3, 4, 5]
      
    },
    {
      "id": 2,
      "name": "Teacher",
      "permissions": [107, 108, 109, 110]
    }
  ],
  "message": "Operation successful"
}
```

### POST /roles/

**Request Body:**

```json
{
  "name": "Teacher",
  "description": "Teaching staff role",
  "department_id": 1,
  "permissions": [107, 108, 109, 110, 112, 113, 114, 115] 
}
```

**Validation Rules:**
- Validate all permission IDs are integers
- Validate all permission IDs are in range (1-180)
- Remove duplicates automatically
- Sort permission IDs
- Include view permission IDs when granting full access (required for read endpoints that check view)

### PUT /roles/{role_id}

**Request Body:**

Same structure as POST request. Update the role's permissions array with new write permission IDs.

---

## Permission Validation

### Validation Rules

1. **Type Validation**: All permission IDs must be integers (not strings or other types)
2. **Range Validation**: All permission IDs must be between 1 and 180 (inclusive)
3. **Duplicate Removal**: Remove duplicate permission IDs automatically
4. **Sorting**: Sort permission IDs in ascending order for consistency
5. **View Permission Filtering**: Optionally filter out "view" permission IDs since read access is implicit

### Validation Process

When receiving permission IDs from API requests:

1. **Parse Input**: Convert all IDs to integers (handle string numbers)
2. **Filter Invalid**: Remove any IDs that are not integers or outside range (1-180)
3. **Remove Duplicates**: Ensure each ID appears only once
4. **Optional Filter**: Remove "view" permission IDs (IDs ending in base, base+5, base+10, etc.)
5. **Sort**: Sort the final list in ascending order
6. **Store**: Save the validated list to the database

### Error Handling

If invalid permission IDs are provided:
- Return 400 Bad Request with error message listing invalid IDs
- Do not partially update permissions
- Provide clear error messages indicating which IDs are invalid

---

## Permission Logic

### Determining Access

**Read Access (View/List) - Implicit:**
- If role has **any permission** for a module/submodule, read access is granted
- No explicit "view" permission ID is required
- Check: Does the role have any permission ID that maps to this module/submodule?
- **Scope**: Basic data viewing, listing, searching, exporting

**Write Access (Create/Edit/Delete) - Explicit:**
- Requires explicit permission ID for the specific action
- Check: Does the role have the specific permission ID for this action?
- **Create**: Check for "create" permission ID
- **Edit**: Check for "edit" permission ID
- **Delete**: Check for "delete" permission ID
- **Scope**: Standard CRUD operations on individual records

**Manage Access (Administrative) - Explicit:**
- Requires explicit "manage" permission ID
- Check: Does the role have the "manage" permission ID for this module/submodule?
- **Scope**: Administrative functions, bulk operations, system configuration, advanced features
- **Different from Read**: Manage allows system-level changes, not just viewing
- **Different from Create/Edit/Delete**: Manage covers administrative operations beyond basic CRUD

### Permission Checking Logic

**For Read Operations:**
1. Identify the module_id and submodule_id for the resource
2. Check if role has any permission ID that maps to this module/submodule
3. If yes → Allow read access
4. If no → Deny access

**For Write Operations:**
1. Identify the module_id, submodule_id, and action (create/edit/delete/manage)
2. Calculate the expected permission ID for this combination
3. Check if role has this specific permission ID
4. If yes → Allow write operation
5. If no → Deny write operation

### Module Access Detection

To determine if a role has access to a module:

1. Get all permission IDs from the role
2. Map each permission ID to its (module_id, submodule_id, action)
3. Extract unique module_ids from the mapped permissions
4. These are the modules the role has access to (with implicit read access)

---

## Examples

### Example 1: Admin Role (Full Access)

**Role Configuration:**
- Name: "Admin"
- Permissions: All write permissions (create, edit, delete, manage) for all modules
- Permission IDs: [2, 3, 4, 5, 7, 8, 9, 10, 12, 13, 14, 15, 17, 18, 19, 20, ...] (all non-view IDs)

**Access Level:**
- ✅ Read access to all modules (implicit)
- ✅ Create access to all modules (explicit permissions)
- ✅ Edit access to all modules (explicit permissions)
- ✅ Delete access to all modules (explicit permissions)
- ✅ Manage access to all modules (explicit permissions)

### Example 2: Teacher Role (Class Management Only)

**Role Configuration:**
- Name: "Teacher"
- Permissions: Write permissions for Class Management module only (module 7: Classes & Sections, Subjects, Assignments, Class Attendance)
- Permission IDs: [112, 113, 114, 115, 117, 118, 119, 120, 122, 123, 124, 125, 127, 128, 129, 130] (create, edit, delete, manage for each of the four submodules; view IDs 111, 116, 121, 126 can be included for explicit read)

**Access Level:**
- ✅ Read access to Class Management module (implicit - has permissions for this module)
- ✅ Create access to Class Management submodules (explicit permissions)
- ✅ Edit access to Class Management submodules (explicit permissions)
- ✅ Delete access to Class Management submodules (explicit permissions)
- ✅ Manage access to Class Management submodules (explicit permissions)
- ❌ No access to other modules (no permissions for other modules)

### Example 3: Read-Only Role

**Role Configuration:**
- Name: "Viewer"
- Permissions: Empty array `[]`
- Permission IDs: None

**Access Level:**
- ❌ No access to any modules (no permissions = no module access)

**Note**: To grant read-only access, you would need to assign at least one permission ID for each module (even if it's a "view" permission ID, though read access is implicit). However, since read access is implicit when module access exists, you could assign a single "create" permission ID to grant read access without write access.

### Example 4: Limited Write Access Role

**Role Configuration:**
- Name: "Editor"
- Permissions: Create and edit permissions only (no delete/manage)
- Permission IDs: [7, 8, 12, 13, 17, 18, ...] (create and edit IDs only)

**Access Level:**
- ✅ Read access to modules with permissions (implicit)
- ✅ Create access (explicit permissions)
- ✅ Edit access (explicit permissions)
- ❌ Delete access (no delete permission IDs)
- ❌ Manage access (no manage permission IDs)

### Example 5: Module Access Without Write Permissions

**Role Configuration:**
- Name: "Reader"
- Permissions: Only "view" permission IDs (though not required)
- Permission IDs: [1, 6, 11, 16, ...] (view IDs only)

**Access Level:**
- ✅ Read access to modules (implicit - has permissions for these modules)
- ❌ No create access (no create permission IDs)
- ❌ No edit access (no edit permission IDs)
- ❌ No delete access (no delete permission IDs)
- ❌ No manage access (no manage permission IDs)

**Note**: Since read access is implicit, you could achieve the same result by assigning a single "create" permission ID for each module, which would grant read access without actually allowing creation (if you check for explicit create permission before allowing creation).

### Example 6: Manage Permission vs Read Access

**Role Configuration:**
- Name: "Data Viewer"
- Permissions: Only "create" permission IDs (grants read access)
- Permission IDs: [7, 12, 17, ...] (create IDs only)

**Access Level:**
- ✅ Read access to modules (implicit - has permissions for these modules)
- ✅ Create access (has create permission IDs)
- ❌ No edit access (no edit permission IDs)
- ❌ No delete access (no delete permission IDs)
- ❌ No manage access (no manage permission IDs)

**What User Can Do:**
- ✅ View student lists and details
- ✅ Create new student records
- ✅ Export student data
- ❌ Cannot edit existing students
- ❌ Cannot delete students
- ❌ Cannot bulk import students (requires manage)
- ❌ Cannot configure student module settings (requires manage)
- ❌ Cannot assign roles/permissions (requires manage)

**Role Configuration:**
- Name: "Administrator"
- Permissions: "manage" permission IDs for all modules
- Permission IDs: [5, 10, 15, 20, 25, 30, 35, ...] (manage IDs only)

**Access Level:**
- ✅ Read access to modules (implicit - has permissions for these modules)
- ❌ No create access (no create permission IDs)
- ❌ No edit access (no edit permission IDs)
- ❌ No delete access (no delete permission IDs)
- ✅ Manage access (has manage permission IDs)

**What User Can Do:**
- ✅ View all data (implicit read access)
- ✅ Configure module settings
- ✅ Bulk import/export data
- ✅ Assign roles and permissions
- ✅ Approve workflows and requests
- ✅ Manage system configurations
- ❌ Cannot create individual records (needs create permission)
- ❌ Cannot edit individual records (needs edit permission)
- ❌ Cannot delete individual records (needs delete permission)

**Key Insight**: "Manage" permission is for **administrative and system-level operations**, not for basic data viewing (which is implicit) or standard CRUD operations (which require specific create/edit/delete permissions).

---

## Best Practices

### 1. Permission Storage

✅ **DO:**
- Store permissions as JSON array of integers: `[1, 2, 3, ...]`
- Always return permissions as array (never null)
- Store all five actions (view, create, edit, delete, manage) for full-access roles so read endpoints that check view work
- Remove duplicates before saving
- Sort permission IDs for consistency
- Validate all IDs before saving to database

❌ **DON'T:**
- Store permissions as strings: `["1", "2", "3"]`
- Store permission objects instead of IDs
- Omit view permission IDs for roles that need to access read endpoints protected by `check_staff_permission(..., action="view")`
- Allow invalid permission IDs (> 180 or < 1)
- Store null instead of empty array `[]`

### 2. Permission Assignment

✅ **DO:**
- Assign permission IDs based on required write actions only
- Remember that module access grants implicit read access
- Validate permission IDs before assignment
- Provide clear error messages for invalid IDs
- Use PermissionRegistry helper for validation

❌ **DON'T:**
- Assign "view" permission IDs unnecessarily (read is implicit)
- Assign permission IDs outside valid range (1-180)
- Allow duplicate permission IDs
- Skip validation when updating permissions

### 3. API Implementation

✅ **DO:**
- Return permissions as integer array in role response
- Return empty array `[]` if role has no write permissions
- Validate permission IDs in request body
- Provide clear error messages for validation failures
- Support both camelCase and snake_case in requests/responses

❌ **DON'T:**
- Return permission objects instead of IDs
- Return null for permissions field
- Accept string permission IDs in requests
- Create a separate `/permissions` endpoint (not needed)
- Omit view permission IDs for full-access roles (needed for read endpoints that check view)

### 4. Access Control Logic

✅ **DO:**
- Check for module access (any permission for module) for read operations
- Check for specific permission ID for write operations
- Use PermissionRegistry to map IDs to module/submodule/action
- Cache permission mappings for performance
- Log permission checks for debugging

❌ **DON'T:**
- Require "view" permission ID for read access
- Check for module access when checking write permissions
- Assume all permissions grant all access levels
- Skip permission checks in API endpoints

### 5. Database Design

✅ **DO:**
- Use JSON column type for permissions
- Set default value to empty array `[]`
- Create index on department_id for faster lookups
- Use database transactions when updating permissions
- Backup permissions data regularly

❌ **DON'T:**
- Create separate permissions table
- Create junction table for role-permissions
- Store permissions as comma-separated string
- Allow null values in permissions column
- Skip database constraints

### 6. Error Handling

✅ **DO:**
- Validate permission IDs before database operations
- Return clear error messages for invalid IDs
- Handle database errors gracefully
- Log permission validation failures
- Provide helpful error messages to API consumers

❌ **DON'T:**
- Silently ignore invalid permission IDs
- Allow partial permission updates on validation failure
- Expose internal error details to API consumers
- Skip error handling in permission operations

---

## Validation Checklist

Before deploying, ensure:

- [ ] Role model uses JSON column type for `permissions` field
- [ ] Permission IDs are stored as integers (not strings) in JSON array
- [ ] Full-access roles include view permission IDs (1–180) so read endpoints that check view work
- [ ] Role API returns `permissions` as integer array: `[1, 2, 3, ...]`
- [ ] All permission IDs in roles are validated (1-180)
- [ ] Permission validation is implemented using PermissionRegistry
- [ ] Empty permissions return as `[]` (not `null`)
- [ ] Database migration creates `roles` table with JSON column
- [ ] Index on `department_id` is created for performance
- [ ] API response format matches examples
- [ ] Access control logic checks module access for read operations
- [ ] Access control logic checks specific permission IDs for write operations
- [ ] Unit tests cover permission validation and role operations
- [ ] Unit tests cover read/write access logic

---

## Summary

**Key Requirements:**
1. ✅ **No separate permissions table** - permissions stored as JSON in `Role.permissions` column
2. ✅ **Module access = read access** - If role has any permission for a module, read access is implicit
3. ✅ **Explicit permissions = write access** - Only create, edit, delete, manage require explicit permission IDs
4. ✅ **180 permission IDs** (1-180) - IDs assigned in specific order (Module → Submodule → Action)
5. ✅ **SQLAlchemy JSON column** - Use `Column(JSON)` type for storing permission arrays
6. ✅ **Integer IDs only** - Store as `[1, 2, 3]` not `["1", "2", "3"]`
7. ✅ **Validation required** - Use PermissionRegistry to validate IDs before saving
8. ✅ **Role API response** - Include `permissions` as integer array in role response (only write permissions)
9. ✅ **No separate permissions API** - Permissions are embedded in role response

**Implementation Approach:**
- **Backend**: Store permissions as JSON array of integers in `Role.permissions` column
- **Read Access**: Implicit when role has any permission for a module/submodule
- **Write Access**: Requires explicit permission ID for specific action (create/edit/delete/manage)
- **Validation**: Use PermissionRegistry helper class to validate permission IDs (1-180)
- **API**: Return permissions array directly from role model (no joins needed)
- **Frontend**: Generates all permissions using PermissionRegistry and matches role's permission IDs

**Benefits:**
- ✅ Simpler schema (no junction table, no separate permissions table)
- ✅ Faster queries (no joins required)
- ✅ Easier to manage (single column update)
- ✅ Flexible (easy to add/remove permissions)
- ✅ Clear separation: module access = read, explicit permissions = write

This ensures seamless integration between backend and frontend permission systems with a clear distinction between read and write access.
