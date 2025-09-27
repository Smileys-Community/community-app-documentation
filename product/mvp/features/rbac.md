# Feature Requirements — RBAC (MVP)

## Roles
- Guest, Member, Group Admin, Superadmin, Event Host (scoped per event).

## Permissions (MVP)
- Guest: read public groups/events.
- Member: join groups (open), RSVP, view own tickets.
- Group Admin: CRUD events for owned group; view RSVPs; check-in.
- Superadmin: create groups; assign admins; platform-level audit.
- Event Host: scan/check-in for assigned events.

## Acceptance Criteria
- Server-side authorization checks for all protected endpoints.
- UI hides unauthorized actions.
- Audit log for: group create/update; role grants; event publish/cancel; check-in reversals.
