---
title: Subjects — How It Works
type: module
project: sms-ui
module: Class Management
last-updated: 2026-03-18
---

# Subjects – How It Works

This document describes how the Subjects feature works in the SMS app: main Subjects view, Copy Subjects, Edit subject, Unassigned subjects, and related APIs.

---

## 1. Overview

- **Subjects** are curriculum items (e.g. Mathematics, Language II). They can be **assigned** to a **class** and **section** (class–section). One subject can be assigned to many class–sections.
- **Subject options**: A subject can have **options** (e.g. Language II → French, German, Spanish). Options are used when the same subject name has multiple choices per class–section.
- **Two main screens**:
  - **Subjects** (`SubjectsView`): Manage subjects **by class and section**. You pick a class and section, then see/create/edit/remove/copy subjects for that class–section.
  - **Unassigned subjects** (`UnassignedSubjectsView`): List subjects that are **not assigned to any** class–section. Used to delete subjects that are no longer needed.

---

## 2. Subjects View (Class & Section)

**Location:** Class Management → Subjects (with class/section selectors).

### 2.1 Loading classes and sections

- On open, the app loads **classes** via `GlobalService.ensureClassesLoaded()` (GET `/classes/`) and shows the app **LoadingService** overlay until the response is received.
- When the user selects a **class**, **sections** are loaded via `GlobalService.ensureSectionsLoaded()` (GET `/sections/`) with the same LoadingService overlay.
- Class and Section dropdowns show hint "Loading..." and are disabled while loading. No separate inline spinner; loading is indicated by the app overlay only.

### 2.2 Loading subjects for a class–section

- Subjects for the **selected class and section** are loaded only when **both** are selected.
- On section change (and when class + section are selected), the presenter calls `loadSubjects(classId, sectionId)`:
  - Shows LoadingService overlay.
  - Calls **ClassSubjectRepository.getSubjectsByClass(classId, sectionId: sectionId)** → GET `/classes/{classId}/subjects?section_id={sectionId}`.
  - Response is a grouped list (subject + teachers per row); the repository flattens it to `List<ClassSubject>` and the view displays it.
- The list is stored in `_classSubjects` and filtered by search into `_filteredClassSubjects`.

### 2.3 List behaviour

- **Search:** Filters by subject name, code, description, or teacher name.
- **Sort:** Name (A–Z, Z–A), Date created (oldest/newest first).
- **Row actions (menu):** Edit, Delete (Remove from class). Delete is only shown when `classSubject.id != 0` (i.e. the row is a real class–subject link; rows with `id == 0` are not removable from this screen).
- **Display:** Each row shows subject name, code, description, teacher (from StaffListController), and type (Mandatory / Not markable). Subject data is resolved from **GlobalService** by `subjectId`.

---

## 3. Create Subject (Inline)

- Shown only when a **class and section** are selected and user has create permission.
- User can type a name and optionally choose **subject type** (Mandatory / Not markable) and **Has options**.
- On submit, **SubjectPresenter.createSubject()**:
  - Creates the subject via **SubjectRepository** (POST `/subjects/`) with name, code, description, type, hasOptions.
  - Then assigns it to the current class–section via **ClassSubjectRepository.assignSubjectToClass()** (POST `/classes/{classId}/subjects` with `subject_id`, `section_id`).
- Loading is shown via presenter’s `showLoading()` / `hideLoading()` (LoadingService overlay). List refreshes after success.

---

## 4. Edit Subject (Drawer)

- **Open:** Row menu → Edit. Opens the **end drawer** with `_SubjectEditDrawerContent`.
- **Content:** Subject name, code, description, type, has options. For subjects **with options**, a list of options (e.g. French, German) with Add / Edit / Delete.
- **Save (subject master data):** Calls **SubjectRepository.updateSubject()** (PUT `/subjects/{id}`). Then presenter calls `loadSubjects()` for the current class–section to refresh the list. Loading via LoadingService.
- **Option actions** (add/edit/delete option):
  - Add: POST `/subjects/{id}/options`
  - Edit: PUT `/subjects/{id}/options/{optionId}`
  - Delete: DELETE `/subjects/{id}/options/{optionId}`
- While an option API is in progress, the drawer shows an overlay (`_optionActionLoading` + `AppLoading.overlay`) so loading is visible until the response is received.

---

## 5. Remove from Class / Delete

- **Remove from class:** Row menu → Delete. Confirmation dialog “Remove from Class”. Calls **SubjectPresenter.deleteSubject(classSubjectId, subjectId, false, classId, sectionId)**:
  - **ClassSubjectRepository.removeSubjectFromClass(classSubjectId)** → DELETE `/class-subjects/{classSubjectId}`.
  - Then `loadSubjects(classId, sectionId)` to refresh the list. Loading via LoadingService.
- **Delete subject entirely:** Not done from the main Subjects list; only “Remove from class” is offered here. Full delete is used from Unassigned subjects (see below). If the backend supported “delete from all classes”, the same presenter method with `deleteFromAllClasses: true` would call **SubjectRepository.deleteSubject(subjectId)** (DELETE `/subjects/{id}`) and remove the subject from GlobalService.

---

## 6. Copy Subjects

Copy assigns **existing subjects** from a **source** class (–section) to the **target** class–section (the one currently selected on the main screen).

### 6.1 Opening the copy flow

- User must select **target** class (and section) first on the main Subjects screen.
- Button **“Copy from Class”** opens the **Copy Subjects** end drawer (same slot as Edit drawer; only one is visible at a time).
- Drawer UI matches the staff profile history drawer: fixed width (400), header with icon and title, then body.

### 6.2 Copy drawer UI

- **Header:** “Copy Subjects” and:
  - **Copying from:** Source class name and, if a source section is chosen, section name; otherwise “All Sections”.
  - **Copying to:** Current target class and section (from main screen).
- **Source selection:**
  - **Source Class** dropdown (from GlobalService classes).
  - **Source Section** dropdown (optional): “All Sections” or a specific section for that class. If “All Sections”, source subjects are not loaded (section is required to fetch source subjects).
- **Source subject list:**
  - Loaded when **Source Class** and **Source Section** are both set (section not null).
  - **ClassSubjectRepository.getSubjectsByClass(sourceClassId, sectionId: sourceSectionId)** → GET `/classes/{id}/subjects?section_id={sectionId}`.
  - While this request is in progress, the drawer shows **AppLoading.inline()** (no list yet). When the response is received, the list is shown.
  - **Deduplication by subject:** For subjects with options (e.g. Language II with French, German, Spanish), the API returns one row per option. The view **deduplicates by subjectId** and merges options so that **one row per subject** is shown with a line like “Options: French, German, Spanish”. Selection is per **subject** (subjectId), not per option.
- **Selection:** Each source subject row has an **AppSwitch**. The selection state is stored in `_selectedSubjectsForCopy` (map `subjectId → bool`). User toggles which subjects to copy.
- **Actions:** “Copy” runs the copy API; “Close” closes the drawer.

### 6.3 Copy API and loading

- **Copy** is enabled only when there is a source class, target class and section, and at least one subject selected.
- On **Copy**, **SubjectPresenter.copySubjects()**:
  - Shows LoadingService overlay.
  - Calls **ClassSubjectRepository.copySubjects()**:
    - POST `/classes/{targetClassId}/subjects/copy`
    - Body: `source_class_id`, `source_section_id` (optional), `target_class_id`, `target_section_id`, `subject_ids`, `copy_teachers: false`.
  - On success, calls `loadSubjects(targetClassId, targetSectionId)` to refresh the target list, shows success message, and closes the drawer.
  - Hides loading in `finally`.

So both “load source list” and “copy” use visible loading (inline in drawer for source list, app overlay for copy) until the response is received.

---

## 7. Unassigned Subjects View

- **Route:** Class Management → Subjects → Unassigned subjects (e.g. `/{schoolId}/class-management/subjects/unassigned-subjects`).
- **Purpose:** List subjects that are **not assigned to any** class–section. Used to clean up unused subjects.
- **Load:** **SubjectRepository.listUnassignedSubjects()** → GET `/subjects/unassigned`. While loading or deleting, the view shows **AppLoading.overlay** so loading is visible for the whole request.
- **List:** Each row shows subject name, code, type. User can select multiple and delete.
- **Delete:** Confirmation dialog then **SubjectRepository.deleteSubject(id)** (DELETE `/subjects/{id}`). GlobalService is updated (remove subject). List is reloaded. Loading overlay is shown during delete (`_isDeleting`).

---

## 8. APIs Used (Summary)

| Purpose | API / Repository method | HTTP |
|--------|---------------------------|------|
| List classes | GlobalService / ClassRepository | GET `/classes/` |
| List sections | GlobalService / SectionRepository | GET `/sections/` |
| Subjects for class–section | ClassSubjectRepository.getSubjectsByClass | GET `/classes/{id}/subjects?section_id={id}` |
| Create subject | SubjectRepository.createSubject | POST `/subjects/` |
| Assign to class | ClassSubjectRepository.assignSubjectToClass | POST `/classes/{id}/subjects` |
| Update subject | SubjectRepository.updateSubject | PUT `/subjects/{id}` |
| Remove from class | ClassSubjectRepository.removeSubjectFromClass | DELETE `/class-subjects/{id}` |
| Delete subject | SubjectRepository.deleteSubject | DELETE `/subjects/{id}` |
| List unassigned | SubjectRepository.listUnassignedSubjects | GET `/subjects/unassigned` |
| Copy subjects | ClassSubjectRepository.copySubjects | POST `/classes/{targetId}/subjects/copy` |
| Subject options (list) | SubjectOptionRepository | GET `/subjects/{id}/options` |
| Add option | SubjectOptionRepository | POST `/subjects/{id}/options` |
| Update option | SubjectOptionRepository | PUT `/subjects/{id}/options/{optId}` |
| Delete option | SubjectOptionRepository | DELETE `/subjects/{id}/options/{optId}` |

---

## 9. Loading and Cursor Rules

- Every API that changes data or loads a list shows a **visible** loading indicator from **start of request until response** (success or error):
  - **App LoadingService** overlay for: load classes, load sections, load subjects (class–section), create subject, update subject, remove from class, copy subjects, and option add/edit/delete in the edit drawer.
  - **Drawer-scoped:** Copy drawer shows **AppLoading.inline()** while loading the **source** subject list; copy action itself uses LoadingService overlay. Edit drawer shows overlay when `_optionActionLoading` for option APIs.
- Unassigned view uses overlay for list load and for delete. Disabling a control without a visible indicator is not used as the only loading signal.

---

## 10. Permissions

- Subject actions are gated by **ModuleConfig.moduleClassManagement**, **submoduleSubjects** (submoduleSubjects):
  - **create:** Create subject, Copy subjects.
  - **edit:** Edit subject (and options).
  - **delete:** Remove from class; delete subject (Unassigned view).
- Permission checks are done in the presenter or view before calling the API; missing permission shows an error or hides the action.

---

## 11. Key Files

- **Views:** `sms/lib/views/subjects_view.dart`, `sms/lib/views/unassigned_subjects_view.dart`
- **Presenter:** `sms/lib/presenters/subject_presenter.dart`
- **Repositories:** `sms/lib/repositories/subject_repository.dart`, `sms/lib/repositories/class_subject_repository.dart`, `sms/lib/repositories/subject_option_repository.dart`
- **Models:** `sms/lib/models/subject_model.dart`, `sms/lib/models/class_subject_model.dart`, `sms/lib/models/subject_option_model.dart`
- **Routing:** `sms/lib/core/routing/app_router.dart` (subjects and unassigned-subjects routes)
