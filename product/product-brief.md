# Product Brief
A mobile-first community app where people discover groups, join, RSVP to events, and get verified at the door with QR tickets. Admins create groups/events and manage attendance safely.


- **Primary Goals (MVP):**
  - Guests can browse public groups and events without login.
  - Members can join groups and RSVP to events.
  - Organizers can publish events and verify attendance on-site.

## MVP
- Public landing + directories (Groups, Events).
- Auth: email/password, Google & Facebook.
- RBAC: Guest, Member, Group Admin, Superadmin, Event Host (event-scoped).
- Groups & Events CRUD (admins), public event page with map.
- RSVP + QR ticket issuance; email confirmations & reminders.
- Ticket scanning + check-in (web camera).
- Profile: My Groups, My Events, Settings (email, password, delete).
- Legal pages, OG/meta for sharing, basic analytics, mobile-first UI.

## Out of Scope (MVP → later)
- Private groups & invite links, reputation/badges, abuse reporting UI,
- Personalized feeds, search, chat/DM, push notifications, vetting workflow.

## Users & Personas
- **Guest:** browses without login.
- **Member:** joins groups, RSVPs, holds tickets.
- **Group Admin:** manages a group & events.
- **Event Host:** scans tickets and checks in attendees.
- **Superadmin:** platform ops, assigns admins.

## High-Level User Journeys
- **Discovery → RSVP:** Guest sees event → Sign up → Join group → RSVP → Ticket email → Attend → Scan & check-in.
- **Admin flow:** Superadmin creates group → assigns Group Admin → Admin creates event → publishes → attendees RSVP → Host scans.

## Functional Requirements (summary)
- Public pages with searchable lists; event detail page with map and OG metadata.
- Auth with email verification to RSVP; social login unifies by email.
- RBAC enforced server-side; UI hides unauthorized actions.
- Event creation with title/description (markdown), schedule, venue (Maps URL), capacity (optional).
- RSVP requires membership in host group; one ticket per RSVP; tickets are unique and verifiable.
- Ticket scanning: camera access in browser; idempotent check-in; online verification.

## Non-Functional Requirements
- **Perf:** p95 API < 400ms; ticket check-in endpoint p95 < 250ms.
- **Availability:** 99.9% for auth/RSVP/check-in.
- **Security:** JWT access + refresh, OWASP mitigations, TLS, audit logs for admin actions.
- **Privacy:** minimal PII, encrypt at rest, delete/export on request, cookie consent for analytics.
- **Observability:** structured logs, tracing for auth/RSVP/check-in, dashboards.

## Analytics
- Events: view_event, search_events, rsvp_start/success, ticket_view, ticket_scanned,
  signup_start/success, login_success, group_joined, event_created/published.
- Dashboards: Verified attendances/week, view→RSVP→check-in funnel, email deliverability.

## Dependencies & Integrations
- Email provider (DKIM/SPF/DMARC), Google Maps URL/embed, OAuth (Google, Facebook).

##  Risks & Mitigations
- **Spam/low-quality events:** RBAC; manual group admin curation; email verification.
- **Onsite scanning latency:** lean endpoint + caching; pre-warm; offline fallback (v1.x).
- **Email deliverability:** verified domain; suppression of bounces/complaints.

## 12) Acceptance / Launch Criteria
- A user can complete the full Guest→Member→RSVP→Ticket→Check-in flow on mobile.
- Admins can create/publish events and see live check-ins.
- Core analytics and email deliverability verified in production.