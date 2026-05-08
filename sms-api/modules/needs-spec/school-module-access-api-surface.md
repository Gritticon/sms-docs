---
title: "School module access — sms-api"
type: planning-stub
project: sms-api
module: school module access
last-updated: null
---

## Status

**Needs spec** — Implementation detected in codebase during pipeline sync on 2026-04-25, but no `docs/sms-api/` file references the `school_module_access` route module (zero filename matches under `docs/sms-api/` at time of scan).

## What was found in code

- **File(s):** `sms-api/app/api/routes/school_module_access.py`
- **Type:** FastAPI — exposes two routers registered in `sms-api/app/main.py`: `internal_router` (prefix `/internal/schools`, internal employee auth) and `school_router` (prefix `/schools`, staff/school context).
- **Role:** Per-school enablement of modules and submodules (`SchoolModuleAccess` model, `ModuleAccessItem` / `SubmoduleAccessItem` schemas in `app/schemas/school_module_access_schema.py`).

## What needs to be documented

- [ ] Full API contract (list/get/update module flags; bulk vs per-module)
- [ ] Internal vs school-facing endpoint matrix and which JWT each uses
- [ ] Interaction with KYC gating and `kyc_school` / app `isModuleEnabled` behavior
- [ ] Audit and change history expectations for support

## Open questions

- Is internal-only surface documented for Gritticon admin only, or also schools?
- Default module set for new school onboarding (see onboarding checklist)
