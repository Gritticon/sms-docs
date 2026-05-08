# KYC School Settings Gating Regression

## Goal

Ensure module visibility is always controlled by school settings, including login, reload, and direct URL navigation.

## Scope

- `kyc/lib/app/kyc_app.dart`
- `kyc/lib/router/app_router.dart`
- `kyc/lib/views/home/home_shell_view.dart`
- `kyc/lib/views/home/kyc_authenticated_shell.dart`

## Baseline Test Data

- School A: `fees=false`, `orders=false`, all others `true`
- School B: all modules `true`
- One valid student account per school

## Manual Regression Matrix

### 1) Login flow gating

- Open `/:schoolId/login` for School A and login with valid student.
- Expected:
  - Brief loading state while school settings are fetched.
  - Hidden modules (`fees`, `orders`) are not visible in home/account/rail.
  - Enabled modules remain visible.

### 2) Hard reload gating

- While logged in to School A on `/:schoolId/home`, do browser hard reload.
- Expected:
  - Loading state appears before module UI.
  - Hidden modules do not flash at any time.
  - Final UI still hides `fees` and `orders`.

### 3) Direct URL guard (disabled module)

- While logged in to School A, open:
  - `/:schoolId/fees`
  - `/:schoolId/orders`
- Expected:
  - Router redirects to `/:schoolId/home`.
  - Disabled page never becomes interactive.

### 4) Direct URL allow (enabled module)

- While logged in to School A, open:
  - `/:schoolId/attendance`
- Expected:
  - Route opens normally (no redirect).

### 5) School switch isolation

- Login to School A, then logout and login to School B.
- Expected:
  - No stale module filtering from School A.
  - School B modules reflect School B settings only.

## Quick Smoke Checklist (release)

- [ ] Login shows settings-driven modules only.
- [ ] Reload does not show all modules even briefly.
- [ ] Disabled module deep links redirect to home.
- [ ] Enabled module deep links continue to work.
- [ ] Cross-school logout/login does not leak prior settings.

## Notes

- This spec intentionally prioritizes **settings fetch before module rendering**.
- Any future route addition under `/:schoolId/*` must be reviewed for module gating in router redirect logic.
