# Cursor Implementation Prompt — Payroll Module Redesign

> Copy everything below the divider into Cursor. Do not modify it before pasting.
> Feed it phase by phase — complete Phase 1 fully before starting Phase 2.

---

## PROMPT

You are implementing the full payroll module redesign for a Flutter Web school management system called **Gritticon SMS**. This is a desktop-first, admin-facing ERP.

---

### STEP 0 — READ BEFORE WRITING ANY CODE

Read these files fully before touching any code:

1. `docs/product/planned/payroll-ui-redesign-v2.md` — the complete product plan. Every screen, every field, every interaction, every data model change, every API endpoint, and the full implementation order is defined there. Do not guess or invent anything that is not in this document.
2. `sms-ui/lib/models/payroll_run_model.dart`
3. `sms-ui/lib/models/payslip_model.dart`
4. `sms-ui/lib/repositories/payroll_repository.dart`
5. `sms-ui/lib/controllers/payroll_controller.dart`
6. `sms-ui/lib/views/payroll_workspace_view.dart`
7. `sms-ui/lib/core/routing/app_router.dart`
8. `sms-ui/lib/core/theme/app_colors.dart`

Do not start writing code until you have read all of the above.

---

### PROJECT CONTEXT

- **Stack:** Flutter Web (Dart), GetX for state management, GoRouter for navigation, MVP pattern (Presenter + View interface)
- **Pattern to follow:** Look at `lib/controllers/staff_list_controller.dart`, `lib/repositories/staff_repository.dart`, and `lib/presenters/staff_presenter.dart` as reference for how a complete module is structured. Follow the same pattern exactly.
- **State:** All reactive state uses `Rx<T>` / `.obs` / `RxList` / `RxBool` from GetX. Controllers extend `GetxController`.
- **Navigation:** GoRouter with deferred imports for code splitting. Add new routes to `app_router.dart` following the existing deferred import pattern.
- **Colors:** Use only values from `lib/core/theme/app_colors.dart`. Do not hardcode hex values in widget files.
- **Naming convention:** snake_case for file names. PascalCase for class names. Suffix: `_view.dart`, `_controller.dart`, `_repository.dart`, `_model.dart`, `_widget.dart`.

---

### GUARD RAILS — READ CAREFULLY

1. **Follow the plan exactly.** `payroll-ui-redesign-v2.md` is the single source of truth. If something is not in the plan, do not add it.
2. **No auto-LOP.** LOP is never auto-calculated. The drafter adds it manually as a deduction line item.
3. **No "Reviewed" step.** The lifecycle is `draft → finalized` only. There is no reviewed status, no `reviewed_by` field, no review endpoint.
4. **Template changes do not affect open drafts.** Template → draft is a one-way copy at generation time.
5. **Patch, don't reload.** After adding or removing a single line item, patch only that payslip in state. Do not reload the full run.
6. **One draft per month.** Enforce at the API call level: if a draft already exists for the chosen month, show an inline validation error in the New Draft dialog. Do not create a second draft for the same month.
7. **Settings do not live on the payroll home screen.** Settings (Earnings & Deductions, Templates, Assign Templates, Payroll Settings) are only accessible via the ⚙ icon navigating to a separate settings section.
8. **Finalized views are fully read-only.** No `[ × ]` buttons, no `[ + Add ]`, no Generate panel, no Delete button.
9. **Destructive actions always require a confirmation dialog** before proceeding: delete draft, delete earning/deduction master, delete template, recalculate (resets manual changes), finalize.
10. **Currency format:** Always display rupee amounts as `₹X,XX,XXX` (Indian number system). Use tabular figures in table columns so amounts align.

---

### PHASE 1 — BACKEND (FastAPI, `sms-api`)

Implement steps 1–17 from the Implementation Order in the plan. Each step must be independently testable before moving to the next.

**Start with:** Read `sms-api/` to understand the existing payroll service, models, and routes before making any changes.

Key pointers:
- Existing payroll service: find files with "payroll" in `sms-api/` — read them before modifying.
- DB migrations: create Alembic migrations for every schema change. Do not alter tables manually.
- For the `earning_masters.is_basic` flag: enforce at the API layer that only one earning per school can have `is_basic = true`. If a new earning is set as basic, unset the previous one automatically and return a warning in the response.
- Remove the entire auto-LOP block from `calculate_payroll()` — set `lop_days = 0`, `lop_amount = 0`, `lop_breakdown = []` at generation.
- The finalize endpoint must NOT require `status = reviewed`. It finalizes from `draft` directly.
- New `GET /payroll/runs/drafts` returns all runs with `status = draft`, no pagination.
- New `GET /payroll/runs/finalized` supports `limit` and `offset` params (default `limit=3`).
- `POST /payroll/runs` body must include `name` (string, required).

Complete all 17 backend steps and verify each endpoint returns correct data before moving to Phase 2.

---

### PHASE 2 — FLUTTER UI (`sms-ui`)

Implement steps 18–32 from the Implementation Order in the plan. Build in the listed order — each widget/view depends on the one before it.

**Start with:** Read the existing payroll view files (listed in Step 0) to understand what already exists. You are replacing, not building from scratch.

**File-by-file instructions:**

**Step 18 — `payroll_run_model.dart`**
- Add `name` field (String)
- Change `status` enum to only `draft` and `finalized` (remove `reviewed`)
- Remove `reviewedBy` and `reviewedAt` fields

**Step 19 — `payslip_model.dart`**
- Add `PayslipLineItem` class: `{id, label, amount, isFromTemplate}`
- Add attendance fields: `daysPresent`, `daysAbsent`, `daysOnLeave`, `daysHoliday`
- Add getter `attendanceSummary` → `'{daysPresent}P  {daysAbsent}A  {daysOnLeave}L  · {total} days'`
- Change `earningItems` and `deductionItems` to `List<PayslipLineItem>`
- Change `status` enum to only `draft` and `finalized`

**Step 20 — `payroll_repository.dart`**
- Add `fetchDrafts()` → `List<PayrollRun>`
- Add `fetchFinalized({int limit, int offset})` → `List<PayrollRun>`
- Add `createRun({required String name, required String dateFrom, required String dateTo})` → `PayrollRun`
- Add `finalizeRun(String runId)` → `PayrollRun`
- Add `deleteRun(String runId)` → void
- Add `removePayslipFromRun(String runId, String payslipId)` → void
- Add `addStaffToRun(String runId, String staffId)` → `Payslip`
- Add `addLineItem(String payslipId, String itemType, String label, double amount)` → `Payslip`
- Add `removeLineItem(String payslipId, String lineId)` → `Payslip`
- Add earning masters CRUD, deduction masters CRUD, templates CRUD, staff-templates methods

**Step 21 — `payroll_controller.dart`**
- `RxList<PayrollRun> drafts` — all draft runs
- `RxList<PayrollRun> finalizedRuns` — loaded finalizations (start with 3)
- `RxBool hasMoreFinalized` — whether more finalizations exist to load
- `loadHome()` — fetches drafts and first 3 finalizations in parallel
- `loadMoreFinalized()` — appends next 3 to `finalizedRuns`
- `createDraft(name, dateFrom, dateTo)` — validates month uniqueness, creates, navigates to draft detail
- `deleteDraft(runId)` — removes from `drafts` list
- `finalizeDraft(runId)` — finalizes, moves run from `drafts` to `finalizedRuns`

**Step 22 — `payroll_home_view.dart`**
- Layout per Screen 1 spec in the plan
- Two sections: Drafts (with New Draft button) and Finalized Payrolls (with Load next 3)
- ⚙ icon top-right → navigate to settings section
- Empty states for both sections

**Step 23 — `payroll_new_draft_dialog.dart`**
- Modal dialog (not a full page)
- Month + year pickers (dropdowns)
- Draft name field (auto-fills from month selection, editable)
- Date range fields (auto-fill from month, editable)
- Inline validation: if draft for that month already exists, show error below month picker and disable Create button
- "Create Draft" → calls controller → on success, navigate to draft detail

**Step 24 — `payroll_line_item_cell_widget.dart`**
- Takes: `List<PayslipLineItem> items`, `String cellType` (earnings/deductions), `bool isEditable`
- Renders each item as one row: `label   ₹amount   [ × ]`
- Custom items (isFromTemplate = false) render label in secondary text color
- Divider + Total row below items
- When `isEditable = true`: shows `[ + Add ]` below total
- `[ + Add ]` expands an inline form: Label field + Amount field + ✓ Add + Cancel
- `[ × ]` on item triggers `onRemoveItem(lineId)` callback
- On add: triggers `onAddItem(label, amount)` callback
- No API calls inside — all callbacks bubble up to controller

**Step 25 — `payroll_staff_row_widget.dart`**
- One table row for a payslip
- Shows: Emp ID | Staff Name + Designation | Attendance summary | Earnings cell | Deductions cell | Net Total | × (if editable)
- Net Total = `payslip.grossSalary - payslip.totalDeductions`, formatted in Indian currency
- Uses `PayrollLineItemCellWidget` for Earnings and Deductions
- Row height: expands to fit the cell with most line items
- `×` row button triggers `onRemoveStaff(payslipId)` callback

**Step 26 — `payroll_draft_table_widget.dart`**
- Full-width table
- Column headers: Emp ID | Staff Name | Attendance | Earnings | Deductions | Net Total | (×)
- Rows: `ListView` of `PayrollStaffRowWidget`
- Grand Total footer row (right-aligned, bold)
- `[ + Add Staff ]` button (bottom-left)
- `isEditable` bool — when false, hides all × buttons and Add buttons (used for finalized view)
- `[ + Add Staff ]` opens a staff search dialog (searchable list of active staff not in this draft)

**Step 27 — `payroll_draft_detail_view.dart`**
- Full page, route: `/payroll/drafts/:runId`
- Header: back arrow, draft name (editable inline), date range, status chip, Delete Draft button, Finalize button
- Generate/Recalculate panel (spec in plan — shows last generated time if already generated)
- Warning banners (collapsible) for: no-template staff, unfinalized attendance, missing settings
- `PayrollDraftTableWidget(isEditable: true)`
- Finalize triggers confirmation dialog → on confirm → controller finalizes → navigate to home

**Step 28 — `payroll_finalized_view.dart`**
- Full page, route: `/payroll/finalized/:runId`
- Same layout as draft detail but: no generate panel, no banners, no Delete button, no Finalize button
- Header shows: "Finalized on {date} by {name}"
- `PayrollDraftTableWidget(isEditable: false)` — all controls hidden

**Step 29 — `payroll_settings_config_view.dart`**
- Payroll Settings sub-page per Screen 8 spec
- Fields: working days (numeric), rounding rule (dropdown), cutoff day (numeric), payslip visibility (toggle)
- Save button with loading state and success toast

**Step 30 — `payroll_earnings_masters_view.dart`**
- Two tabs: Earnings | Deductions
- Each tab: sortable table + Add button + Edit/Delete per row
- Inline or modal forms for add/edit
- Delete warns if component is used in any template

**Step 31 — `payroll_templates_view.dart`**
- Templates list view: table with name, component counts, staff count, Edit/Delete
- Template detail view (separate route or side panel): name field, earnings component picker, deductions component picker, reorder (up/down), remove (×)
- Delete blocked if any staff are assigned — show error message

**Step 32 — `payroll_assign_templates_view.dart`**
- Staff table: Emp ID, Name, Department, Assigned Template (or "No template" badge), Change/Assign action
- Filters: Department dropdown, Template dropdown
- Per-row assignment: click Change → template picker inline
- Bulk: checkboxes → "Assign Template to N selected staff" button → template picker → confirm

**Step 33 — `payroll_settings_view.dart`**
- Settings section shell
- Left sidebar navigation with 4 items: Earnings & Deductions | Templates | Assign Templates | Payroll Settings
- Highlighted active item
- Right content area renders the selected sub-page

---

### PHASE 3 — ROUTING

**Step 34 — Update `app_router.dart`**

Add the following routes using deferred imports (same pattern as existing payroll routes):

| Route path | View file | Notes |
|---|---|---|
| `/payroll` | `payroll_home_view.dart` | Replace existing payroll home |
| `/payroll/drafts/:runId` | `payroll_draft_detail_view.dart` | |
| `/payroll/finalized/:runId` | `payroll_finalized_view.dart` | |
| `/payroll/settings` | `payroll_settings_view.dart` | |
| `/payroll/settings/earnings` | handled inside settings shell | default sub-page |
| `/payroll/settings/templates` | handled inside settings shell | |
| `/payroll/settings/assign` | handled inside settings shell | |
| `/payroll/settings/config` | handled inside settings shell | |

The settings sub-pages are managed inside `PayrollSettingsView` as an internal navigation state — no separate top-level routes needed for them.

---

### PHASE 4 — INTEGRATION TESTING

**Step 35 — Test full flows in this order:**

1. **Settings setup:** Add 3 earnings → Add 2 deductions → Create a template using them → Assign template to 5 staff → Verify assigned count shows on templates list
2. **Create draft:** Open payroll → New Draft → April 2026 → Create → verify navigates to draft detail page
3. **Generate:** Click Generate → verify all 5 assigned staff appear with correct earnings/deductions → verify attendance summary shows for each
4. **Line item add:** Add a custom deduction "LOP 2 days ₹500" to one staff → verify net total updates immediately → verify grand total updates
5. **Line item remove:** Remove "HRA" from one staff → verify earnings total and net total update immediately
6. **Add staff:** Click + Add Staff → add a staff member → verify their template items populate and attendance shows
7. **Remove staff:** Remove a staff member → verify grand total decreases
8. **Finalize:** Click Finalize → confirm dialog → finalize → verify navigates to home → verify run appears in Finalized section → verify it no longer appears in Drafts
9. **Finalized view:** Click View on the finalized run → verify table is read-only (no × buttons, no Add buttons, no Generate panel)
10. **Load more:** Create and finalize 4+ runs → verify Home shows 3 → click Load next 3 → verify 3 more appear
11. **Month uniqueness:** Try to create a second draft for April 2026 → verify inline error appears and Create button is disabled
12. **Delete draft:** Create a draft → delete it → verify it is removed from the list

---

### WHAT NOT TO DO

- Do not add any features not in the plan document
- Do not add auto-LOP calculation anywhere
- Do not add a "Reviewed" status, button, or endpoint
- Do not let template changes silently update open drafts
- Do not do a full run reload after a single line item add/remove — patch only the changed payslip
- Do not hardcode color hex values in widget files — use `AppColors.*`
- Do not create new files unless the plan calls for them
- Do not break existing non-payroll modules

---

### WHEN IN DOUBT

Re-read `docs/product/planned/payroll-ui-redesign-v2.md`. Every screen, field, interaction, and edge case is documented there. If something is still unclear after reading the plan, ask before implementing.
