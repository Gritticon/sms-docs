---
title: Payroll Module — Full UI Redesign v2
type: planned
status: planned
project: platform
module: Payroll
last-updated: 2026-04-13
supersedes: payroll-redesign-plan.md, payroll-draft-redesign-plan.md
---

# Payroll Module — Full UI Redesign v2
### Cursor Implementation Instructions | No code, step-by-step

## Context

This plan supersedes both `payroll-redesign-plan.md` and `payroll-draft-redesign-plan.md`.
It incorporates all prior decisions and adds a redesigned module structure based on
the following confirmed decisions:

| Decision | Choice |
|---|---|
| Multiple drafts | Multiple drafts can coexist across different months (one per month max) |
| Draft lifecycle | Draft → Finalized directly. No "Reviewed" step. |
| Payroll home | Shows all drafts + last 3 finalizations with "Load next 3" |
| Draft page | Full separate page; table with inline earnings/deductions per staff row |
| Attendance in draft | Inline summary per staff row: `22P  3A  1L  · 26 days` |
| LOP | Not auto-calculated. Drafter adds manually as a deduction line item. |
| Earnings & Deductions | Master records managed in Settings. Grouped into Templates. Templates assigned to staff. |
| Settings location | Separate Settings section, not tabs on the home screen |
| Template → draft | One-way copy at generation time. Template changes don't affect open drafts. |

---

## Prior Plan — What to Keep vs What Changes

### Keep from `payroll-redesign-plan.md`
- All schema changes: Part 2 (with modifications noted below)
- Backend pipeline: Part 3 (with LOP removed per draft redesign plan)
- Bug fixes: Part 1 (all 5 fixes)
- API endpoints: Part 4 (with modifications noted below)

### Keep from `payroll-draft-redesign-plan.md`
- Draft table design (inline breakdown cells per staff row)
- Attendance summary format and source logic
- Earnings/deductions JSON format with `id` and `is_from_template`
- `PayslipLineItem` model
- All 4 draft mutation endpoints (remove staff, add staff, add line item, remove line item)

### What Changes in This Plan
- PayrollRun status enum: remove `reviewed`; only `draft` and `finalized`
- PayrollRun: add `name` field
- Remove `reviewed_by` and `reviewed_at` fields from PayrollRun
- Payslip status enum: remove `reviewed`; only `draft` and `finalized`
- Remove review endpoint (`PUT /payroll/{run_id}/review`)
- Home screen: drafts list + finalized list (no month/year navigation)
- Settings: separate section with 4 sub-pages
- Finalize endpoint: no longer requires `status = reviewed` — can finalize from `draft`

---

## Module Structure

```
Payroll (module root)
├── Home                    ← default view when opening Payroll
│   ├── Drafts section      ← all drafts (any month)
│   └── Finalized section   ← last 3 runs, load more
├── Draft Detail Page       ← /payroll/drafts/:run_id
├── Finalized View Page     ← /payroll/finalized/:run_id (read-only)
└── Settings                ← gear icon from home
    ├── Earnings & Deductions
    ├── Templates
    ← Assign Templates
    └── Payroll Settings
```

---

## Screen 1 — Payroll Home

### Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  Payroll                                          [⚙ Settings]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DRAFTS                                        [ + New Draft ]  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  April 2026 Payroll          26 staff · Created 10 Apr   │   │
│  │  01 Apr 2026 – 30 Apr 2026                 [Open] [🗑]   │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │  March 2026 Payroll          24 staff · Created 28 Mar   │   │
│  │  01 Mar 2026 – 31 Mar 2026                 [Open] [🗑]   │   │
│  └──────────────────────────────────────────────────────────┘   │
│  (empty state if no drafts)                                      │
│                                                                  │
│  ──────────────────────────────────────────────────────────     │
│                                                                  │
│  FINALIZED PAYROLLS                                              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  February 2026   24 staff   ₹5,48,200   Finalized 05 Mar │   │
│  │                                                  [View]   │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │  January 2026    24 staff   ₹5,44,800   Finalized 04 Feb │   │
│  │                                                  [View]   │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │  December 2025   22 staff   ₹5,10,000   Finalized 03 Jan │   │
│  │                                                  [View]   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              [ Load next 3 ]                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Drafts Section Spec

- Section header: "DRAFTS" (label) + "New Draft" button (right-aligned)
- Each draft card displays:
  - Draft name (e.g., "April 2026 Payroll")
  - Date range: "01 Apr 2026 – 30 Apr 2026"
  - Staff count (from number of payslips in the run, 0 if not yet generated)
  - Created date: "Created 10 Apr"
  - **Open** button → navigates to Draft Detail Page
  - **Delete** icon → confirmation dialog: _"Delete 'April 2026 Payroll'? This cannot be undone."_
- Empty state: "No drafts yet. Create a new draft to start working on payroll."

### Finalized Section Spec

- Section header: "FINALIZED PAYROLLS"
- Each finalized card displays:
  - Month and Year
  - Staff count
  - Total net salary (e.g., ₹5,48,200)
  - Finalized date
  - **View** button → navigates to Finalized View Page (read-only)
- Initially shows last 3 by `finalized_at` descending
- "Load next 3" button loads the next 3; hide button when all are loaded
- Empty state: "No finalized payrolls yet."

### Settings Icon

- Gear icon (⚙) top-right of the Payroll home screen
- Navigates to Settings section (separate route)

---

## Screen 2 — New Draft Dialog

Triggered by "New Draft" button. Renders as a modal dialog (not a new page).

```
┌─────────────────────────────────┐
│  New Payroll Draft          [×] │
├─────────────────────────────────┤
│  Draft Name                     │
│  [April 2026 Payroll        ]   │
│                                 │
│  Month                          │
│  [April ▾]  [2026 ▾]           │
│                                 │
│  Date Range                     │
│  [01 Apr 2026]  to  [30 Apr 2026]│
│  (auto-fills from month; editable)│
│                                 │
│  [Cancel]        [Create Draft] │
└─────────────────────────────────┘
```

### Behaviour

- **Draft Name**: auto-fills as "{{Month}} {{Year}} Payroll" from month selection; admin can edit
- **Month picker**: month dropdown + year dropdown (or a single month-year picker)
- **Date range**: auto-fills first and last day of selected month; editable for non-standard periods (e.g., mid-month payroll)
- **Validation**: if a draft already exists for the selected month, show inline error below the month picker:
  _"A draft already exists for April 2026. Delete it first to create a new one."_
  Disable the "Create Draft" button in this case.
- On "Create Draft": POST to create run → navigate directly to the new Draft Detail Page

---

## Screen 3 — Draft Detail Page

Full page. Navigated to from "Open" on a draft card, or immediately after "Create Draft".

### Header

```
← Payroll     April 2026 Payroll      01 Apr – 30 Apr 2026     [Draft]
                                        [Delete Draft]  [Finalize]
```

- Back arrow "← Payroll" → returns to Home (no changes to save; draft auto-saves on every mutation)
- Draft name (editable inline — click to edit)
- Date range display (read-only after creation)
- Status chip: "Draft" (blue/neutral)
- **Delete Draft** button: text button, destructive color — confirmation dialog:
  _"Delete 'April 2026 Payroll'? All unsaved changes will be lost."_
- **Finalize** button: primary — see Finalize Flow below

### Generate / Recalculate Panel

```
┌──────────────────────────────────────────────────────────────┐
│ Generate draft from salary templates and attendance data.     │
│                                    [ Generate Draft ]        │
└──────────────────────────────────────────────────────────────┘
```

If already generated (payslips exist):
```
┌──────────────────────────────────────────────────────────────┐
│ Last generated: 10 Apr 2026, 2:15 PM  ·  26 staff loaded     │
│ ⚠ Recalculate will reset all manual changes.                  │
│                                    [ Recalculate ]           │
└──────────────────────────────────────────────────────────────┘
```

- "Recalculate" shows a confirmation dialog before proceeding:
  _"Recalculate draft? All manual line-item changes and custom entries will be reset."_

### Warning Banners

Show relevant banners between the generate panel and the table. Each is collapsible.

| Condition | Banner text |
|---|---|
| X staff have no template | "3 staff have no salary template assigned — they will not appear in the draft." |
| Attendance not finalized | "Attendance not finalized for 5 staff. Attendance summary may be incomplete." |
| Payroll settings missing | "Payroll settings not configured. Using defaults (26 working days, nearest rounding)." |

### Draft Table

The main content of the page. Full-width, horizontally scrollable if needed.

```
Emp ID  │  Staff Name      │  Attendance            │  Earnings              │  Deductions            │  Net Total  │  ×
────────┼──────────────────┼────────────────────────┼────────────────────────┼────────────────────────┼─────────────┼───
EMP001  │  Ravi Kumar      │  22P  3A  1L  · 26d    │  Basic         ₹18,000  ×  │  PF            ₹ 2,160  ×  │             │
        │  Mathematics     │                         │  HRA           ₹ 7,200  ×  │  ESI           ₹   213  ×  │  ₹27,127    │  ×
        │                  │                         │  Transport     ₹ 1,500  ×  │  ─────────────────────     │             │
        │                  │                         │  ──────────────────────    │  Total         ₹ 2,373     │             │
        │                  │                         │  Total         ₹26,700     │  [ + Add ]                 │             │
        │                  │                         │  [ + Add ]                 │                            │             │
────────┼──────────────────┼────────────────────────┼────────────────────────────┼────────────────────────────┼─────────────┼───
EMP002  │  Priya Sharma    │  26P  0A  0L  · 26d    │  Basic         ₹22,000  ×  │  PF            ₹ 2,640  ×  │             │
        │  English         │                         │  HRA           ₹ 8,800  ×  │  ─────────────────────     │  ₹29,312    │  ×
        │                  │                         │  ──────────────────────    │  Total         ₹ 2,888     │             │
        │                  │                         │  Total         ₹30,800     │  [ + Add ]                 │             │
        │                  │                         │  [ + Add ]                 │                            │             │
────────┴──────────────────┴────────────────────────┴────────────────────────────┴────────────────────────────┴─────────────┴───
                                                                                                  Grand Total  ₹56,439
[ + Add Staff ]
```

### Column Definitions

| Column | Source | Behaviour |
|---|---|---|
| Emp ID | `staff.employee_id` | Read-only |
| Staff Name + Designation | `staff.name`, `staff.designation` | Read-only |
| Attendance | `staff_attendance` records for the draft date range | `{present}P  {absent}A  {leave}L  · {working_days} days` — read-only display |
| Earnings | Copied from template at generation | Line items with × per item; total row; [ + Add ] |
| Deductions | Copied from template at generation | Same as earnings |
| Net Total | `sum(earnings) − sum(deductions)` | Recalculates live on every add/remove |
| × (row) | Remove this staff from the draft | Confirmation: _"Remove Ravi Kumar from this draft?"_ |

### Earnings / Deductions Cell Detail

Each line item renders as one row inside the cell:

```
Basic Salary     ₹ 18,000   [ × ]    ← from template
HRA              ₹  7,200   [ × ]    ← from template
LOP 3 days       ₹    621   [ × ]    ← drafter added (shown with subtle different style)
──────────────────────────────────
Total            ₹ 24,579
                 [ + Add ]
```

- `[ × ]` on a line item: removes it from this payslip only. Template is not changed.
- Custom items (drafter-added) are visually distinguishable — e.g., lighter label color or "custom" badge.
- `[ + Add ]` opens an inline input below the list:
  ```
  Label [________________]  Amount [₹ ______]   [✓ Add]  [Cancel]
  ```
  On ✓ Add: API call → patch the payslip in state (no full reload).
- Net Total recalculates immediately after every add/remove.

### [ + Add Staff ] Footer

- Opens a search dialog listing all active staff not currently in this draft.
- Staff picker shows: name, designation, department, assigned template name (or "No template" badge).
- On select: generates payslip from template + fetches attendance → adds row to table.

### Finalize Flow

1. Admin clicks "Finalize"
2. Confirmation dialog:
   ```
   Finalize April 2026 Payroll?

   26 staff  ·  Total Net ₹5,48,200

   This will lock all payslips. No changes can be made after finalization.

   [Cancel]    [Finalize]
   ```
3. On confirm: API call → run status set to `finalized` → navigate back to Payroll Home
4. The run now appears in the Finalized section on Home.

---

## Screen 4 — Finalized Run View (Read-Only)

Same layout as Draft Detail Page with the following differences:

- Header status chip: "Finalized" (green)
- "Finalized on 05 Mar 2026 by Meena Sharma" subtitle
- No "Delete Draft" button
- No "Finalize" button
- No "Generate / Recalculate" panel
- No `[ × ]` on line items (all read-only)
- No `[ + Add ]` buttons
- No `[ + Add Staff ]` footer
- No `[ × ]` per row
- Grand total shown same as draft view

---

## Screen 5 — Settings: Earnings & Deductions

Accessible from: Settings section → "Earnings & Deductions"

### Layout

Two tabs: **Earnings** | **Deductions**

---

**Earnings tab:**

```
Earnings                                         [ + Add Earning ]

Name              Type               Value / Rate    Is Basic    Actions
──────────────────────────────────────────────────────────────────────
Basic Salary      Fixed              ₹ 18,000        ● (yes)     [Edit] [Delete]
HRA               % of Basic         50%             —           [Edit] [Delete]
Transport Allow.  Fixed              ₹  1,500        —           [Edit] [Delete]
Special Allow.    % of Gross         10%             —           [Edit] [Delete]
```

**Add/Edit Earning form (inline or modal):**
- Name (text)
- Type: Fixed | % of Gross | % of Basic
- Value: if Fixed → amount (₹); if % → rate (%)
- Is Basic: toggle (only one earning can have this on; enforce at API: if another has `is_basic=true`, auto-disable it and warn admin)

**Delete behaviour:** If the earning is used in any template, show warning:
_"Basic Salary is used in 4 templates. Removing it will remove it from those templates."_
Require explicit confirmation.

---

**Deductions tab:**

```
Deductions                                       [ + Add Deduction ]

Name          Calc Base    Rate/Amount    Cap         Employer %    Actions
────────────────────────────────────────────────────────────────────────────
PF            Basic        12%            —           12%           [Edit] [Delete]
ESI           Gross        0.75%          ₹ 1,800     3.25%         [Edit] [Delete]
Prof Tax      Fixed        ₹ 200          —           —             [Edit] [Delete]
```

**Add/Edit Deduction form:**
- Name (text)
- Calculation Base: Fixed | % of Basic | % of Gross
- Rate/Amount: amount (₹) or rate (%)
- Cap Amount: optional (₹) — deduction will not exceed this
- Employer Contribution %: optional — employer's share (used for display; does not affect net salary)

**Delete behaviour:** same as earnings — warn if used in templates.

---

## Screen 6 — Settings: Templates

### List View

```
Templates                                        [ + New Template ]

Name                    Earnings    Deductions    Staff     Actions
────────────────────────────────────────────────────────────────────
Teaching Staff          4           3             18        [Edit] [Delete]
Non-Teaching Staff      3           2             8         [Edit] [Delete]
Admin Staff             3           3             4         [Edit] [Delete]
```

- Each row: template name, count of earnings components, count of deduction components, count of assigned staff
- **Edit** → opens Template Detail Page
- **Delete** → if staff are assigned: _"4 staff are assigned to this template. Reassign them before deleting."_ Block deletion if staff are assigned.

### Template Detail Page

```
← Templates     Teaching Staff Template

EARNINGS                                         [ + Add Earning Component ]
────────────────────────────────────────────────────────────────────────────
Basic Salary      Fixed      ₹ 18,000                           [↑] [↓] [×]
HRA               % Basic    50%                                [↑] [↓] [×]
Transport Allow.  Fixed      ₹ 1,500                            [↑] [↓] [×]
Special Allow.    % Gross    10%                                [↑] [↓] [×]

DEDUCTIONS                                       [ + Add Deduction Component ]
────────────────────────────────────────────────────────────────────────────
PF                % Basic    12%      Cap: —    Employer: 12%   [↑] [↓] [×]
ESI               % Gross    0.75%    Cap: ₹1,800  Employer: 3.25%  [↑] [↓] [×]
Prof Tax          Fixed      ₹ 200    Cap: —    Employer: —     [↑] [↓] [×]
```

- "Add Earning Component" → picker from Earnings masters (only shows items not already added)
- "Add Deduction Component" → picker from Deductions masters
- `[↑] [↓]` arrows reorder the component (order is preserved in template and shown in draft)
- `[×]` removes the component from this template (does not delete the master)
- No inline value editing — values come from the master records. If admin needs to change a value (e.g., different Basic for different roles), they should create a separate earning master entry.
- Template name is editable inline at the top.
- Auto-save on every change (no explicit Save button needed).

---

## Screen 7 — Settings: Assign Templates

### Layout

Staff table with template assignment column. This is the single place to manage who is on which template.

```
Assign Templates

Search [______________]   Department [All ▾]   Template [All ▾]

  │  Staff Name          │  Department        │  Assigned Template       │  Action
──┼──────────────────────┼────────────────────┼──────────────────────────┼──────────
☐ │  Ravi Kumar          │  Mathematics       │  Teaching Staff          │  [Change]
☐ │  Priya Sharma        │  English           │  Teaching Staff          │  [Change]
☐ │  Anand Mehta         │  Administration    │  — No template           │  [Assign]
☐ │  Lakshmi Rao         │  Support           │  Non-Teaching Staff      │  [Change]
```

- Filter by department and by template
- "No template" rows shown with a warning badge
- **Change / Assign** opens a small dropdown/picker inline: list of all templates → select → confirm
- **Bulk Assign:**
  - Select multiple rows via checkboxes → "Assign Template" button appears at top:
    `[Assign Template to 3 selected staff ▾]` → opens template picker → confirm

### Warning

If a staff member has their template changed while an open draft exists for the current month, the change only affects **future** draft generation. A banner appears:

_"Template change saved. Open drafts are not affected — recalculate the draft to apply the new template."_

---

## Screen 8 — Settings: Payroll Settings

```
Payroll Settings

Working Days per Month
[ 26 ]
Used to calculate per-day rate for LOP and attendance-based deductions.

Rounding Rule
[ Nearest ▾ ]   (Nearest / Round Up / Round Down)

Payroll Cutoff Day
[ 25 ]
Attendance after this day of the month is not included in the current month's draft.

Payslip Visible to Staff
[  ●  ON  ]
When enabled, staff can view and download their own finalized payslips.

[ Save Settings ]
```

- Show a "Settings not configured" banner on Payroll Home until these are saved at least once.
- After saving, show success toast: "Payroll settings saved."

---

## Data Model Changes

### 1. `{school_id}_payroll_runs` table

**Add:**

| Column | Type | Notes |
|---|---|---|
| `name` | varchar(200) | Draft name, e.g., "April 2026 Payroll" |

**Modify:**
- `status` enum: change from `draft`, `reviewed`, `finalized` → `draft`, `finalized`
- Remove `reviewed_by` column
- Remove `reviewed_at` column

**Fix (from prior plan):**
- `finalized_by`: change FK from `internal_employees.id` to plain int referencing school staff

**Keep (from prior plan):**
- `notes`, `total_employees`, `total_gross`, `total_deductions`, `total_net`

---

### 2. `{school_id}_payslips` table

**Modify:**
- `status` enum: change from `draft`, `reviewed`, `finalized` → `draft`, `finalized`

**Keep all fields from prior plans:**
- `working_days`, `paid_days`, `lop_days`, `lop_amount`, `lop_breakdown`
- `days_present`, `days_absent`, `days_on_leave`, `days_holiday`
- `advance_deduction`, `is_amended`, `amended_by`, `amendment_notes`
- `earnings` (JSON array of `PayslipLineItem`)
- `deductions` (JSON array of `PayslipLineItem`)

---

### 3. All other schema changes

Keep as specified in `payroll-redesign-plan.md` Part 2:
- `{school_id}_payroll_settings` (Step 2.1)
- `earning_masters.is_basic` flag (Step 2.2)
- `deduction_masters.employer_contribution_rate` and `is_advance_deduction` (Step 2.3)
- `{school_id}_staff_advances` table (Step 2.6)

---

## API Changes

### Modified Endpoints

| Change | Detail |
|---|---|
| `POST /payroll/runs` | Add `name` field to request body |
| `GET /payroll/runs` | Return `name` and simplified status (`draft`/`finalized`) |
| `PUT /payroll/{run_id}/finalize` | Remove requirement for `status = reviewed`; allow finalize from `draft` directly |
| `GET /payroll/runs/finalized` | New: list finalized runs with pagination (`limit=3`, `offset`) |
| `GET /payroll/runs/drafts` | New: list all draft runs (no pagination — all drafts shown) |

### Remove

- `PUT /payroll/{run_id}/review` — no longer needed (no review step)

### Keep from Prior Plans

All endpoints from `payroll-redesign-plan.md` Part 4, and all 4 draft mutation endpoints from `payroll-draft-redesign-plan.md`:
- `DELETE /payroll/runs/{run_id}/payslips/{payslip_id}`
- `POST /payroll/runs/{run_id}/payslips`
- `POST /payroll/payslips/{payslip_id}/line-items`
- `DELETE /payroll/payslips/{payslip_id}/line-items/{line_id}`

### New Endpoints (Settings)

These are either new or were not clearly split into sub-routes before:

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/payroll/earning-masters` | List all earning master records |
| `POST` | `/payroll/earning-masters` | Create earning master |
| `PUT` | `/payroll/earning-masters/{id}` | Edit earning master |
| `DELETE` | `/payroll/earning-masters/{id}` | Delete (reject if used in a template) |
| `GET` | `/payroll/deduction-masters` | List all deduction master records |
| `POST` | `/payroll/deduction-masters` | Create deduction master |
| `PUT` | `/payroll/deduction-masters/{id}` | Edit deduction master |
| `DELETE` | `/payroll/deduction-masters/{id}` | Delete (reject if used in a template) |
| `GET` | `/payroll/templates` | List all templates with component counts and assigned staff count |
| `POST` | `/payroll/templates` | Create template |
| `PUT` | `/payroll/templates/{id}` | Edit template name / component list |
| `DELETE` | `/payroll/templates/{id}` | Delete (reject if any staff assigned) |
| `GET` | `/payroll/templates/{id}` | Template detail with full component list |
| `GET` | `/payroll/staff-templates` | List all staff with their assigned template |
| `PUT` | `/payroll/staff-templates/{staff_id}` | Assign/change template for a staff member |
| `PUT` | `/payroll/staff-templates/bulk` | Bulk assign: body `{staff_ids: [], template_id}` |

---

## Flutter UI — Files and Components

### New / Rewritten Files

| File | Purpose |
|---|---|
| `payroll_home_view.dart` | Home screen: drafts list + finalized list |
| `payroll_new_draft_dialog.dart` | New Draft modal dialog |
| `payroll_draft_detail_view.dart` | Draft detail full page (table + header + footer) |
| `payroll_finalized_view.dart` | Finalized run read-only view (reuse table widget, all controls hidden) |
| `payroll_draft_table_widget.dart` | Reusable wide table widget (used by both draft and finalized views) |
| `payroll_staff_row_widget.dart` | One staff row with attendance, earnings cell, deductions cell, net |
| `payroll_line_item_cell_widget.dart` | Earnings or Deductions cell with line items, totals, add/remove |
| `payroll_settings_view.dart` | Settings section shell with left sub-nav |
| `payroll_earnings_masters_view.dart` | Earnings & Deductions settings sub-page |
| `payroll_templates_view.dart` | Templates list and detail sub-page |
| `payroll_assign_templates_view.dart` | Assign Templates sub-page |
| `payroll_settings_config_view.dart` | Payroll Settings (working days, rounding, etc.) sub-page |

### Updated Files

| File | Change |
|---|---|
| `payroll_model.dart` | Add `name` to PayrollRun; simplify status enum; add `PayslipLineItem` class |
| `payroll_repository.dart` | Add all new endpoints; update existing run endpoints |
| `payroll_notifier.dart` / provider | Update state to hold drafts list + finalized list separately |

### Design System

Use the following values across the payroll module (data-dense admin UI):

| Token | Value |
|---|---|
| Font - Headings | Inter (or Fira Code for numeric columns) |
| Font - Body | Inter |
| Primary color | #2563EB |
| Background | #F8FAFC |
| Card background | #FFFFFF |
| Text primary | #1E293B |
| Text secondary | #64748B |
| Border | #E2E8F0 |
| Success / Finalized chip | #16A34A |
| Draft chip | #2563EB |
| Warning banner | #FEF3C7 (amber-100) border #F59E0B |
| Destructive | #DC2626 |
| Table row hover | #F1F5F9 |
| Table row height | 56px minimum; expands with line items |
| Custom line item label color | #64748B (secondary, to distinguish from template items) |

### UX Rules for This Module

1. **No full reload after line item changes.** Only patch the single payslip in state. Full reload only after: generate, recalculate, add staff, remove staff.
2. **Auto-save on every mutation.** No explicit Save button on the draft table. The header shows last-mutated timestamp.
3. **Confirm before all destructive actions.** Delete draft, delete earning master, delete template, finalize, recalculate.
4. **Currency formatting.** All rupee amounts use `₹` prefix + comma-separated Indian number format (e.g., ₹1,48,200). Use tabular/monospaced figures in columns so amounts align visually.
5. **Attendance summary is read-only.** Admin cannot edit it in the draft. To fix attendance, they go to the Attendance module.
6. **Settings section is non-destructive to open drafts.** Changing a template, earning, or deduction master does not affect already-generated drafts.

---

## Implementation Order

Run in sequence. Each step is independently testable.

### Phase 1 — Foundation (Backend)

```
1.  Bug fixes from payroll-redesign-plan.md (Fixes 1.1 – 1.5)
2.  DB migrations:
    a. payroll_settings table
    b. earning_masters.is_basic column
    c. deduction_masters.employer_contribution_rate column
    d. staff_advances table
    e. payslip columns: working_days, paid_days, lop_days, lop_amount, lop_breakdown,
       days_present, days_absent, days_on_leave, days_holiday, advance_deduction,
       is_amended, amended_by, amendment_notes, status
    f. payroll_runs: add name; change status to draft/finalized; remove reviewed_by, reviewed_at;
       fix finalized_by FK
3.  Payroll Settings API (GET/PUT /payroll/settings)
4.  Earning Masters CRUD API
5.  Deduction Masters CRUD API
6.  Templates CRUD API (create, edit, delete, get-by-id, list)
7.  Staff Templates API (list, assign, bulk assign)
8.  Updated payroll calculation (Step 3.1 from prior plan — with is_basic fix, no hardcoded rates)
9.  Attendance summary query in calculate_payroll()
10. Advance deduction step (Step 3.3 from prior plan)
11. Remove auto-LOP block; set lop_days = 0 at generation
12. Draft lifecycle: create (with name), finalize (no review requirement)
13. New GET /payroll/runs/drafts endpoint
14. New GET /payroll/runs/finalized endpoint (with limit/offset)
15. 4 draft mutation endpoints (remove staff, add staff, add line item, remove line item)
16. Individual payslip endpoints (get, amend, staff view, PDF — from prior plan)
17. Staff Advances API (from prior plan)
```

### Phase 2 — Flutter UI

```
18. payroll_model.dart — update PayrollRun (add name, fix status), add PayslipLineItem
19. payroll_repository.dart — add all new methods
20. payroll_notifier.dart — separate state for drafts list and finalized list
21. payroll_home_view.dart — drafts section + finalized section + gear icon
22. payroll_new_draft_dialog.dart — month picker, name field, validation
23. payroll_line_item_cell_widget.dart — line items list + add/remove inline
24. payroll_staff_row_widget.dart — one row: attendance + earnings cell + deductions cell + net
25. payroll_draft_table_widget.dart — full table: staff rows + grand total footer + add staff
26. payroll_draft_detail_view.dart — header + generate panel + banners + table
27. payroll_finalized_view.dart — same table in read-only mode
28. payroll_settings_config_view.dart — working days, rounding, cutoff, visibility toggle
29. payroll_earnings_masters_view.dart — earnings + deductions tabs with CRUD
30. payroll_templates_view.dart — templates list + template detail with components
31. payroll_assign_templates_view.dart — staff table with template column + bulk assign
32. payroll_settings_view.dart — settings shell with left sub-nav linking all 4 sub-pages
```

### Phase 3 — Integration Testing

```
33. Full draft flow: Create → Generate → Add/remove line items → Add/remove staff → Finalize
34. Settings flow: Add earning → Add deduction → Create template → Assign to staff → Generate draft → Verify template items appear
35. Finalized view: Verify all edit controls are hidden
36. Load more finalizations: Verify pagination works
37. Advance deduction: Create advance → Generate payroll → Verify deduction appears in payslip row
38. Recalculate: Make manual changes → Recalculate → Verify manual changes are reset
```

---

## Out of Scope (Future)

- Payslip PDF download (planned in prior plan — keep as future scope)
- Staff "My Payslips" screen (planned in prior plan — keep as future scope)
- Statutory compliance reports (Form 16, PF filing export)
- Bulk bank transfer file export
- Arrears, bonus, variable pay automation
- LOP auto-suggestion (system suggests, drafter confirms)
- Bulk add/remove line items across all staff in one action
- Draft export to Excel
