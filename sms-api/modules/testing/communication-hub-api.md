---
title: "Communication Hub API"
type: planning-stub
project: sms-api
module: Communication Hub
last-updated: 2026-04-08
---

> Auto-promoted to testing on 2026-04-08 — implementation detected in codebase. Awaiting QA sign-off before moving to built/.

## Status

**Implemented, not specced, gaps found** — Routes exist under `/communications/*` and `/kyc/messages`,
`/kyc/post`, `/kyc/notifications`. Full gap analysis produced 2026-05-29.
See [`../../../sms-ui/modules/testing/communication-hub-gap-analysis.md`](../../../sms-ui/modules/testing/communication-hub-gap-analysis.md).

Key API gaps: no DELETE/UPDATE endpoints, wrong permission ID on notifications (101→102),
conversation unique constraint missing `target_role`, `_sync_conversations_for_class_section`
N-write storm on every list request, no read/unread endpoint, no push dispatch, no school-wide
broadcast scope.

## What this API needs to cover

### Messages (1-to-1 or 1-to-few, threaded)
- `POST /messages` — Send a message (from: staff, to: studentId/parentId/groupId, body, attachments?)
- `GET /messages/threads` — List message threads for the authenticated user
- `GET /messages/threads/{threadId}` — Get messages in a thread (paginated)
- `POST /messages/threads/{threadId}/reply` — Reply to a thread
- `PUT /messages/{id}/read` — Mark message as read

### Announcements (broadcast, no reply)
- `POST /announcements` — Create announcement (audience: school/class/section/student, body, schedule?)
- `GET /announcements` — List announcements (filter by audience, date)
- `GET /announcements/{id}` — Get announcement detail
- `DELETE /announcements/{id}` — Delete / recall announcement

### Posts (KYC home feed items — richer content)
- `POST /posts` — Create a post (title, body, image?, audience, pin?)
- `GET /posts` — List posts (KYC reads this for home feed, paginated)
- `PUT /posts/{id}` — Edit post
- `DELETE /posts/{id}` — Remove post

### Delivery / Push Notifications
- `POST /notifications/send` — Internal: trigger push notification to a set of device tokens
- `GET /notifications/{userId}` — Get notification history for a user
- `PUT /notifications/{id}/read` — Mark notification as read

## Key decisions to make before writing spec

- Are messages stored in the same DB as other SMS data, or a separate messaging service?
- Real-time delivery: WebSockets or push-only (FCM/APNs for mobile)?
- Attachment support in v1? (adds storage complexity)
- Audience model: can a teacher message only their class, or any student?
- Read receipts: required in v1?

## Related docs

- SMS-UI stub: `sms-ui/modules/needs-spec/communication-hub`
- KYC spec: `/doc/kyc/specs/built/communication-spec`
- KYC notifications spec: `/doc/kyc/specs/built/notifications-events-spec`
- API overview: `/doc/api-overview`
