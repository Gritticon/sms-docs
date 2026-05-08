---
title: "Transport Management API"
type: planning-stub
project: sms-api
module: Transport Management
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — No transport endpoints exist in the current SMS-API. SMS-UI (admin manages) and KYC (parent views assignment) both depend on this.

## What this API needs to cover

### Routes
- `POST /transport/routes` — Create route (name, school, stops ordered list)
- `GET /transport/routes` — List routes for a school
- `GET /transport/routes/{id}` — Route detail with stops
- `PUT /transport/routes/{id}` — Update route
- `DELETE /transport/routes/{id}` — Delete route

### Stops
- `POST /transport/routes/{routeId}/stops` — Add stop (name, lat/lng, arrival time, sequence)
- `PUT /transport/stops/{id}` — Update stop
- `DELETE /transport/stops/{id}` — Remove stop

### Vehicles
- `POST /transport/vehicles` — Add vehicle (registration, capacity, make/model)
- `GET /transport/vehicles` — List vehicles
- `PUT /transport/vehicles/{id}` — Update vehicle
- `DELETE /transport/vehicles/{id}` — Remove vehicle
- `PUT /transport/vehicles/{id}/route` — Assign vehicle to a route

### Drivers
- `POST /transport/drivers` — Add driver (name, contact, licence number)
- `GET /transport/drivers` — List drivers
- `PUT /transport/drivers/{id}` — Update driver
- `PUT /transport/drivers/{id}/vehicle` — Assign driver to vehicle

### Student Assignments
- `POST /transport/assignments` — Assign student to route + stop
- `GET /transport/assignments/student/{studentId}` — Get student's transport assignment
- `GET /transport/assignments/route/{routeId}` — List all students on a route
- `DELETE /transport/assignments/{studentId}` — Remove student from transport
- `POST /transport/assignments/bulk` — Bulk assign by class/section to a route

### Phase 2 — Live Tracking (defer)
- `POST /transport/tracking/location` — Driver app posts GPS coordinates
- `GET /transport/tracking/live/{routeId}` — Live bus location for a route (WebSocket or polling)

## Related docs

- SMS-UI stub: `sms-ui/modules/needs-spec/transport-management`
- KYC spec: `/doc/kyc/specs/built/transport-bus-tracking-spec`
- API overview: `/doc/api-overview`
