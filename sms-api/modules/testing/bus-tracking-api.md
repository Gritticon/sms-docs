---
title: "Bus Tracking API (Phase 2)"
type: planning-stub
project: sms-api
module: Transport Management
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Needs spec** — Phase 2 feature. The KYC spec for bus tracking exists but is marked as deferred. The Transport Management API (routes, vehicles, drivers) is a separate stub — this specifically covers **live GPS tracking**.

## What this API needs to cover

### Driver App — Location Push
- `POST /transport/tracking/location` — Driver app posts GPS coordinates (vehicleId, lat, lng, timestamp, speed, heading)
  - Must be lightweight and handle high frequency (every 5–10 seconds)
  - Authentication: driver-specific token, not full staff JWT

### Parent/Student — Live Location (KYC)
- `GET /transport/tracking/live/{routeId}` — Current location of bus on a route
  - Response: {vehicleId, lat, lng, lastUpdated, nextStop, estimatedArrival?}
  - Options: REST polling (simple) or WebSocket push (real-time)

### Location History
- `GET /transport/tracking/history/{vehicleId}` — Trip replay for a date (for school admin review)
- `GET /transport/tracking/trips/{vehicleId}` — List historical trips

## Infrastructure considerations

- GPS data volume is high — separate time-series table or separate data store (InfluxDB, TimescaleDB)?
- Driver app: standalone Flutter app or embedded in KYC app behind a mode switch?
- ETA calculation: simple geofencing / stop proximity, or full routing engine?
- Push to KYC: WebSocket, SSE, or FCM-triggered polling?

## Key decisions to make before writing spec

- Which driver device — school-provided tablet, driver's own phone, or bus-mounted device?
- Accuracy requirement: real-time (<5s latency) or near-real-time (30s polling)?
- Data retention: how long do we keep GPS history?

## Related docs

- KYC spec: `/doc/kyc/specs/built/transport-bus-tracking-spec`
- Transport Management API: `sms-api/modules/needs-spec/transport-management-api`
- System architecture: `/doc/architecture/system-architecture`
- Phase 2 AI roadmap: `/doc/product/phase2-ai-roadmap`
