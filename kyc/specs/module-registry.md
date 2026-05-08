# KYC Module Registry

Canonical mapping between module integer IDs (public API contract) and slugs
(used internally in Flutter nav and JSON config storage).

**Rule:** IDs are permanent. Never reassign. New modules append with next id.

## Registry

| ID | Slug          | Display Name     | Default |
|----|---------------|-----------------|---------|
| 1  | attendance    | Attendance       | enabled |
| 2  | marks         | Marks & Results  | enabled |
| 3  | fees          | Fees             | enabled |
| 4  | timetable     | Timetable        | enabled |
| 5  | diary         | Diary & Homework | enabled |
| 6  | orders        | School Shop      | enabled |
| 7  | messages      | Messages         | enabled |
| 8  | notifications | Notifications    | enabled |
| 9  | complaints    | Complaints       | enabled |
| 10 | documents     | Documents        | enabled |
| 11 | transport     | Transport        | enabled |
| 12 | library       | Library          | enabled |
| 13 | reminders     | Reminders        | enabled |

Modules always visible (never configurable): **home**, **account**, **profile**.

## Public API Contract

Endpoint: `GET /kyc/school/{school_id}` (no auth required)

```json
{
  "school_id": 220599,
  "school_name": "Demo International School",
  "logo_url": null,
  "primary_color": "#7C3AED",
  "secondary_color": null,
  "enabled_modules": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
}
```

- `enabled_modules: null` → all modules enabled (default when not configured)
- `enabled_modules: [1,2,3]` → only those IDs are shown in nav
- IDs sorted ascending in response

## Storage Layer (SMS-side)

Staff configures via SMS-UI → School Management → KYC Settings.
Stored in `customers.kyc_modules_config` as a JSON slug→bool string:

```json
{"attendance": true, "fees": false, "library": true}
```

API layer (`kyc_school.py`) converts slug→bool map to sorted integer ID list
before returning to KYC clients. KYC never sees the raw slug map.

## KYC Client

- `KycSchoolInfo.enabledModuleIds: Set<int>?`
- `isModuleEnabled(slug)` — maps slug → id via `kycModuleIdBySlug` const,
  checks membership. Unknown slug or null set → `true` (safe default).
- Nav filtering in `kyc_authenticated_shell.dart` and `home_shell_view.dart`.

## Color Fields

| Field           | Type       | Storage        | Notes                    |
|-----------------|------------|----------------|--------------------------|
| primary_color   | String(7)  | `#RRGGBB` hex  | Drives Material seedColor |
| secondary_color | String(7)  | `#RRGGBB` hex  | Reserved, not yet used   |

KYC fetches both colors at school-selection step (before login). Theme
rebuilds immediately on school change. Null → default seed `#7C3AED`.
