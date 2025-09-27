/docs/product/features/events-rsvp-tickets.md
# Feature Requirements — Events, RSVP & Tickets (MVP)

## Goals
Enable admins to publish events; members RSVP; generate verifiable QR tickets.

## User Stories
- As Group Admin, I create and publish events.
- As Member, I RSVP and receive a QR ticket via email.
- As Host/Admin, I scan tickets and check-in attendees.

## Data
- title, description_md, start_at, end_at, tz, venue_name, maps_url, capacity?, visibility, image_url, status[draft,published,cancelled].

## Rules
- RSVP requires membership in host group.
- One active RSVP per user per event.
- Capacity optional; if set and full → block RSVP (waitlist out of scope).

## Acceptance Criteria
- Create/publish/cancel endpoints; event detail page public with map embed/link.
- On RSVP: create RSVP, generate unique QR token, send confirmation email with ticket link.
- Ticket: accessible in profile & event page; single scannable code per RSVP.
- Social share metadata (OpenGraph/Twitter) for event page; ICS “Add to calendar”.

## Non-Functional
- Ticket token unguessable UUID/JWT; verify server-side; expire post-event (+grace).
- p95 API < 400ms; email delivery tracked.
