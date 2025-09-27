# Feature Requirements — Groups (MVP)

## Goals
Provide curated containers for events and memberships.

## User Stories
- As Superadmin, I create groups and assign Group Admins.
- As Member, I view group details and upcoming events.

## Data
- name, description_md, location_city, visibility (public), cover_image_url, admins[].

## Acceptance Criteria
- Superadmin can create/update groups; assign/unassign admins.
- Group page shows: description, admins, member count, upcoming events.
- Join action: “open” in MVP (instant membership).

## Non-Functional
- Pagination on group lists; server validation; audit admin changes.
