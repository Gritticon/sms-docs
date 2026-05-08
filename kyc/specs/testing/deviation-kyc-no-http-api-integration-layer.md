---
title: "Deviation: No shared HTTP repository layer — kyc"
type: deviation
project: kyc
module: Networking
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Architectural deviation detected during pipeline sync on 2026-03-21.

## Deviation details

- **File:** `kyc/lib/` (no `lib/repositories/` or `lib/services/`; `presenters/auth_presenter.dart` uses simulated login)
- **Violation type:** Feature directory expected to pair with a repository for API calls; no `package:http` / Dio usage found in the tree.
- **Expected pattern:** `docs/architecture/system-architecture.md` — Flutter clients call FastAPI over HTTP.
- **Actual:** `AuthPresenter.login` documents “real API will replace this”; other modules use controllers with local/mock data patterns.

## Required action

- [ ] Introduce a shared API client + repository layer per feature when backend endpoints are ready.
- [ ] Confirm MVP scope: mock-only until KYC API is wired.

## Reference

- Architecture doc: `docs/architecture/system-architecture.md`
