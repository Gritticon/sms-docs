---
title: "Audit Logging API"
type: planning-stub
project: sms-api
module: Audit Logging
last-updated: 2026-03-23
---

> Auto-promoted to testing on 2026-03-23 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — No unified audit log endpoints exist. The SMS-UI Audit Log module spec exists (planned) but the API that powers it has no spec yet.

## What this API needs to cover

### Writing Audit Events (internal — called by other modules)
- `POST /audit/events` — Record an audit event (actor, action, resource, resourceId, metadata, timestamp, schoolId)
  - This should be called internally by every write operation across SMS-API
  - Must be fast and non-blocking (async write, never block the main request)

### Reading Audit Events (staff-facing via SMS-UI)
- `GET /audit/events` — Query audit log with filters:
  - `actorId` (staff member who made the action)
  - `module` (which module: student, fees, timetable, etc.)
  - `action` (CREATE, UPDATE, DELETE, LOGIN, EXPORT, etc.)
  - `resourceId` (e.g., a specific student ID)
  - `dateFrom` / `dateTo`
  - `schoolId` (Gritticon admin can see across schools)
  - Paginated
- `GET /audit/events/{id}` — Single event detail (full before/after diff for UPDATE events?)
- `GET /audit/export` — Export filtered audit log as CSV

## Data model considerations

```
AuditEvent {
  id: UUID
  schoolId: string
  actorId: string           # staff member
  actorName: string         # denormalised for query without join
  action: enum              # CREATE | UPDATE | DELETE | LOGIN | LOGOUT | EXPORT | STATUS_CHANGE
  module: string            # e.g. "student", "fee", "timetable"
  resourceType: string      # e.g. "Student", "FeePayment"
  resourceId: string
  resourceLabel: string     # denormalised: e.g. student name
  before: JSON | null       # snapshot before change (UPDATE/DELETE)
  after: JSON | null        # snapshot after change (CREATE/UPDATE)
  ipAddress: string
  userAgent: string
  timestamp: datetime
}
```

## Key decisions to make before writing spec

- Store full before/after snapshots (rich but large) or just action metadata (lean)?
- Retention policy (how long are audit logs kept — 1 year? 7 years for compliance?)
- Is audit log immutable (no deletes) or can Gritticon admin purge old records?
- Separate DB table or separate service? (affects write path performance)

## Related docs

- SMS-UI spec: `/doc/sms-ui/modules/planned/audit-log-module`
- Permission implementation: `/doc/architecture/permission-implementation`
- API overview: `/doc/api-overview`
