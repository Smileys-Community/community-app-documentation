# Product Requirements Document (PRD)
Product: Community App (Groups, Events, RSVP & Ticketing)  
Document Owner: Product  
Version: 0.9 (Draft)  
Last Updated: 2025-10-09

## 0. Purpose & Scope
Deliver an MVP mobile-first web application enabling users to discover public groups and events, join groups, RSVP to events, receive QR code tickets, and be checked in onsite. Admins manage groups and events; event hosts verify attendance. This PRD defines scope, requirements, constraints, success criteria, and acceptance for the MVP launch. Non‑MVP/expanded features are explicitly de-scoped.

## 1. Goals & Non-Goals
### 1.1 Primary Goals (MVP)
1. Public discovery of groups and events without authentication.
2. Seamless signup → join group → RSVP → ticket issuance → onsite check-in.
3. Group admins create/manage groups & events; event hosts perform ticket scanning.
4. Reliable, low-latency ticket verification suitable for mobile device usage.
5. Foundational observability, analytics, and security in place for scale-out.

### 1.2 Non-Goals (Future / Post-MVP)
Refer to Out of Scope list; explicitly excluded: private/invite-only groups, advanced search, chat/DM, personalized feeds, push notifications, reputation systems, abuse reporting UI, vetting workflows, offline scanning (except basic fallback concept), monetization features, growth loops.

## 2. Personas & User Types
| Persona | Description | Key Needs |
|---------|-------------|-----------|
| Guest (Unauthenticated) | Browses groups & events publicly | Fast search/list, SEO, event detail clarity |
| Member | Registered user joined to one or more groups | Easy RSVP, ticket access, profile management |
| Group Admin | Manages a single group & its events | CRUD groups/events, view RSVPs, capacity mgmt |
| Event Host | Performs onsite check-in for specific event(s) | Fast scanning, duplicate detection, status feedback |
| Superadmin | Platform operator | Assign roles, moderate content, audit history |

## 3. User Journeys (High-Level)
1. Discovery → RSVP: Guest views event → Sign up → Email verify (if required) → Join group → RSVP → Receive ticket email → Attend → Host scans QR → Check-in success.
2. Group Admin Flow: Superadmin creates or designates group → Admin edits group profile → Creates event (details, schedule, capacity) → Publishes → Members RSVP → Real-time check-in counts.
3. Host Scanning Flow: Host opens scan page → Grants camera access → Scans ticket → Server validates + idempotent check-in → Visual confirmation.

## 4. Scope & Feature Breakdown
Each requirement tagged (FR = Functional Requirement, NFR = Non-Functional Requirement, ANA = Analytics, SEC = Security, OBS = Observability, RBAC = Role Based Access Control, EMAIL = Email). Numeric IDs for traceability.

### 4.1 Public Discovery
FR-001: Public landing page with concise value proposition and navigation to Groups & Events directories.
FR-002: Public Groups directory: paginated list (default sort: most upcoming events; fallback: alphabetical).
FR-003: Public Events directory: paginated list of upcoming events (sorted by start datetime ascending).
FR-004: Simple keyword filter (groups/events) across name + short description (server-side; no fuzzy logic MVP).
FR-005: Public Event Detail page including: title, description (markdown rendered), start/end time (local + timezone abbreviation), venue (Google Maps link or embedded map), capacity (if set & not zero), RSVP button (contextual state), OG/meta tags for share (title, description, image placeholder).
FR-006: Public Group Detail page: name, description (markdown), list of upcoming published events (next 5), Join/Joined state.

### 4.2 Authentication & Accounts
FR-010: Email/password signup with password reset flow (token via email; 60 min expiry).  
FR-011: Social login (Google, Facebook) unified via email (if existing account → link).  
FR-012: Email verification required before RSVP or group join that triggers ticket issuance.  
FR-013: Session management: Access token (short-lived) + Refresh token (HTTP-only secure cookie).  
FR-014: Account deletion (soft-delete; irreversible purge job within 30 days).  
FR-015: Profile settings: update name, email (re-verify on change), password, delete account trigger.

### 4.3 RBAC & Authorization
RBAC-001: Roles: Guest (implicit), Member, Group Admin (per group), Event Host (per event), Superadmin (global).  
RBAC-002: Enforcement server-side; UI hides unauthorized actions but must not rely solely on client logic.  
RBAC-003: Permission Matrix (excerpt):
- Create Group: Superadmin only (MVP).  
- Edit Group: Group Admin of that group or Superadmin.  
- Create Event: Group Admin of associated group or Superadmin.  
- Publish Event: Same as create event.  
- RSVP to Event: Member who is a member of the group & email verified, event published & capacity not exceeded.  
- Scan Tickets: Event Host for that event, Group Admin of owning group, Superadmin.  
- Assign Group Admin: Superadmin only (MVP manual assignment).  
- Assign Event Host: Group Admin or Superadmin.  
RBAC-004: Idempotent permission checks returning standardized error codes (e.g., ERR_FORBIDDEN_ROLE, ERR_CAPACITY_REACHED).  

### 4.4 Groups & Membership
FR-020: Group creation fields: name (unique slug), description (markdown), optional image placeholder, status (active/archived).  
FR-021: Members can request to join by clicking Join; immediate join (no approval flow in MVP).  
FR-022: View My Groups list in authenticated profile area.  
FR-023: Group Admin sees membership count and list (basic pagination).  

### 4.5 Events & RSVP
FR-030: Event creation fields: title, description (markdown), start_datetime (ISO), end_datetime (optional; must be after start), venue_text (freeform) + optional google_maps_url, capacity (nullable, unlimited if null), group_id (required), publish_status (draft/published), visibility (public only MVP).  
FR-031: Publish action sets publish_status=published and surfaces event in public listings if start_datetime >= now and not archived.  
FR-032: Edit restrictions: cannot change group_id after RSVPs exist; cannot reduce capacity below current RSVP count.  
FR-033: RSVP endpoint: creates RSVP record if user is group member, event published, not started, capacity not exceeded, user does not already have RSVP.  
FR-034: On RSVP success, generate unique Ticket (UUID or ULID) and QR payload (opaque token or signed).  
FR-035: Email confirmation with ticket and event details (see Email section).  
FR-036: Member can view My Events list with RSVP statuses (upcoming, past).  
FR-037: Cancel RSVP (if event not started) frees capacity.  
FR-038: Capacity enforcement atomic (prevent race condition oversubscribe).  
FR-039: Event list includes real-time (or near-real-time) RSVP count (cached <=60s).  

### 4.6 Ticketing & Check-In
FR-050: Ticket contains: ticket_id, event_id, user_id (implicit), issue_timestamp, checksum/signature.  
FR-051: QR code encodes either signed token or short opaque code resolvable server-side.  
FR-052: Scan Page: camera permission request, real-time decode (web API/library), displays ticket status: VALID (and first check-in), ALREADY_CHECKED_IN (with timestamp), INVALID (not found / wrong event / revoked).  
FR-053: Check-in endpoint: idempotent; second submission returns already checked-in without error.  
FR-054: Latency target: p95 < 250ms (NFR-PRF-002).  
FR-055: Audit log entry on each successful check-in (user_id, host_id, timestamp).  
FR-056: Offline fallback (deferred) – store as future feature (not in MVP).  

### 4.7 Notifications & Email
EMAIL-001: Transactional email on RSVP confirmation containing: event title, start time (localized), venue, link to ticket page (with embedded QR).  
EMAIL-002: Reminder email 24h before event start (if RSVP still active).  
EMAIL-003: Password reset email with single-use token link.  
EMAIL-004: Email verification link (expires 24h).  
EMAIL-005: Deliverability tracking (bounces, complaints suppression list).  

### 4.8 Profile & Settings
FR-070: Profile page: My Groups, My Events (upcoming & past), Settings.  
FR-071: Settings modifications require current password (except name).  
FR-072: Display downloadable ticket (QR) and status (Checked-In vs Not Yet).  

### 4.9 Observability & Analytics
OBS-001: Structured JSON logging (request_id, user_id if present, role, endpoint, latency_ms, status_code).  
OBS-002: Distributed tracing spans: auth flows, RSVP, ticket check-in.  
OBS-003: Metrics: http_request_latency, rsvp_count, check_in_count, email_send_success/failure.  
ANA-001: Event view_event (event_id).  
ANA-002: search_events / search_groups (query, results_count bucketed).  
ANA-003: rsvp_start / rsvp_success (event_id).  
ANA-004: ticket_view (ticket_id, event_id).  
ANA-005: ticket_scanned (ticket_id, success flag, first_check_in flag).  
ANA-006: signup_start / signup_success.  
ANA-007: login_success.  
ANA-008: group_joined (group_id).  
ANA-009: event_created / event_published (event_id).  
ANA-010: funnel dashboard: view_event → rsvp_success → ticket_scanned (derived).  

### 4.10 Security & Privacy
SEC-001: JWT access tokens (15m) + refresh tokens (rotating, 7d) in HttpOnly, Secure cookies.  
SEC-002: Password hashing with Argon2id (or bcrypt cost >=12).  
SEC-003: Rate limiting: login (5/min/IP), RSVP (burst 5/min/user), ticket scan (50/min/host).  
SEC-004: Input validation & OWASP top 10 mitigations (XSS sanitize markdown rendering, CSRF protected via same-site cookies or CSRF token).  
SEC-005: Audit logs: admin actions (role grants, event publish, capacity edits) retained 1 year.  
SEC-006: Data encryption at rest for PII (email, name).  
SEC-007: Right to delete/export: export basic personal data (JSON).  
SEC-008: Minimal PII (collect only name, email).  

### 4.11 Performance & Reliability
NFR-PRF-001: p95 API latency (general) < 400ms.  
NFR-PRF-002: p95 Ticket check-in < 250ms.  
NFR-REL-001: Availability for auth/RSVP/check-in endpoints 99.9% (monthly).  
NFR-REL-002: Error budget tracked; on breach, feature freeze for reliability tasks.  
NFR-SCL-001: System supports 10 concurrent scanning hosts per event and 5 scans/sec peak with <2% duplicate conflict errors.  

### 4.12 Accessibility & UX
NFR-A11Y-001: WCAG 2.1 AA for color contrast and form labeling.  
NFR-A11Y-002: Keyboard navigable critical flows (signup, RSVP).  
NFR-A11Y-003: ARIA live region for scan result feedback.  

### 4.13 Internationalization
FR-I18N-001: MVP English only; all user-facing copy centralized to allow future locale expansion.  

## 5. Data Model (Conceptual)
Entities (simplified):
- User(id, email, email_verified, password_hash?, name, created_at, deleted_at?)
- Group(id, name, slug, description_md, image_url?, created_by, created_at, archived_at?)
- GroupMembership(id, user_id, group_id, role=member, created_at)
- Event(id, group_id, title, description_md, start_datetime, end_datetime?, venue_text, google_maps_url?, capacity?, publish_status, visibility, created_by, published_at?, created_at, updated_at)
- RSVP(id, user_id, event_id, status=active/cancelled, created_at, cancelled_at?)
- Ticket(id, rsvp_id, event_id, code (unique or signed), issued_at, checked_in_at?)
- RoleAssignment(id, user_id, group_id?, event_id?, role_type, assigned_by, assigned_at)
- AuditLog(id, actor_user_id, action_type, entity_type, entity_id, metadata_json, created_at)
- EmailLog(id, user_id?, type, status, provider_message_id, created_at)

Constraints & Indices:
- Unique: group.slug, (user.email), (rsvp.user_id, rsvp.event_id), ticket.code
- Indices: event.start_datetime, ticket.event_id+code, role_assignment.user_id+role_type, audit_log.action_type

Capacity Handling:
Atomic transaction: compare current active RSVP count < capacity before insert; use SELECT ... FOR UPDATE or row-level lock.

## 6. API Contract (High-Level Endpoints)
Notation: (A)uth required, (R)ole-specific.

Public GET:
- GET /api/groups?query=&page=  
- GET /api/groups/{slug}  
- GET /api/events?query=&page=  
- GET /api/events/{id}  

Auth (Member):
- POST /api/auth/signup  
- POST /api/auth/login  
- POST /api/auth/refresh  
- POST /api/auth/logout  
- POST /api/auth/password-reset/request  
- POST /api/auth/password-reset/confirm  
- POST /api/auth/verify-email (trigger) & GET /api/auth/verify-email?token=  

Groups & Membership:
- POST /api/groups/{id}/join (A)  
- GET /api/me/groups (A)  

Events & RSVP:
- POST /api/events (A,R=GroupAdmin)  
- PATCH /api/events/{id} (A,R=GroupAdmin)  
- POST /api/events/{id}/publish (A,R=GroupAdmin)  
- POST /api/events/{id}/rsvp (A)  
- DELETE /api/events/{id}/rsvp (A)  
- GET /api/me/events (A)  

Ticketing & Check-In:
- GET /api/tickets/{ticket_id} (A, owner or privileged)  
- GET /api/events/{id}/scan (A,R=Host) – returns scanning session metadata  
- POST /api/scan (A,R=Host) { code }  

Admin / Role Management:
- POST /api/groups (A,R=Superadmin)  
- POST /api/roles/assign (A,R=Superadmin|GroupAdmin context)  
- GET /api/events/{id}/rsvps (A,R=GroupAdmin|Host)  

## 7. UX & UI Guidelines (Summary)
Mobile-first responsive layout (≥360px width). Primary actions as prominent buttons. Color semantics: Green = success (check-in OK), Amber = already checked-in, Red = invalid. Loading skeletons for directory lists. Use markdown renderer with sanitized allowed tags (p, strong, em, ul, li, a, h1-h3, code inline) to prevent XSS.

## 8. Analytics & Dashboards
KPIs:
1. RSVP Conversion Rate: RSVP_success / unique_event_views.  
2. Attendance Verification Rate: ticket_scanned / rsvp_success.  
3. Event Creation Health: events_published per active group per week.  
4. Email Deliverability: (send_success - bounces) / total_sent.  
5. Time to Check-In: median latency server metric for scan endpoint.  
Dashboards: Funnel (view → rsvp → check-in), Weekly Active Hosts, Capacity Utilization (avg RSVP / capacity for non-unlimited events).

## 9. Dependencies & Integrations
Email Provider (e.g., SES / SendGrid) with SPF/DKIM/DMARC. OAuth (Google, Facebook). Optional embedded Google Maps (static map or link). Logging/metrics stack (e.g., OpenTelemetry + backend exporter). Feature flags (optional simple boolean config file for MVP).

## 10. Risks & Mitigations
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Low-quality/spam events | User trust erosion | Medium | Manual group creation (Superadmin), email verification |
| Scan latency spikes | Poor onsite UX | Medium | Lean endpoint, caching, pre-warm, p95 SLA |
| Oversubscription race | Capacity inaccurate | Medium | Transactional lock on RSVP insert |
| Email deliverability issues | Onboarding blocked | Low-Med | Domain auth, bounce suppression, monitoring |
| Duplicate ticket usage | Fraud | Low | Idempotent check-in + already-checked feedback |
| Unauthorized role escalation | Security breach | Low | Strict RBAC, audit logs, least privilege |

## 11. Open Questions / Assumptions
1. Payment / monetization: deferred (assumption: free events only).  
2. Time zones: assume events stored in UTC; display in user browser local timezone.  
3. Image hosting: placeholder only (no upload MVP).  
4. Email verification timing: required before RSVP creation (enforced).  
5. Multiple hosts per event allowed? (Assumption: yes via multiple Event Host assignments).  
6. Rate limit enforcement mechanism TBD (API gateway vs app-level).  
7. Data export format scope (assumption: JSON with user profile + RSVP & ticket history).  

## 12. Acceptance / Launch Criteria
Mapped to earlier requirement IDs.
1. A user (Guest→Member) can discover an event, sign up, join its group, RSVP, receive ticket email, and be successfully checked in onsite (FR-001..FR-005, FR-010..FR-015, FR-021, FR-033..FR-035, FR-050..FR-053, EMAIL-001).  
2. Group Admin can create and publish an event; edit before RSVPs; view RSVP list and counts (FR-030..FR-033, FR-039).  
3. Event Host scanning p95 latency < 250ms over test sample of ≥200 scans (NFR-PRF-002).  
4. Core analytics events emitted and visible in dashboard (ANA-001..ANA-009).  
5. Security baselines: RBAC enforcement, audit logs for admin actions, email verification gate, rate limits present (RBAC-002..004, SEC-003..005).  
6. Observability: request logs + tracing for auth/RSVP/check-in + metrics (OBS-001..003).  
7. Accessibility basic criteria met for signup & RSVP flows (NFR-A11Y-001..002).  
8. Data deletion export flow functional (SEC-007).  
9. No critical P1 bugs open; all Sev1 incidents resolved within SLA during beta period.  
10. Documentation: Admin & Host quick-start guide created.

## 13. Post-MVP Backlog (Not in Launch Scope)
- Private/invite-only groups & invite links.  
- Advanced search & filtering (multi-facet, location radius).  
- Push notifications / mobile app wrapper.  
- Chat / direct messaging.  
- Reputation, badges, moderation workflows UI.  
- Offline scan mode + sync.  
- Monetization: paid tickets, sponsorship, subscription tiers.  
- Feature flags system robust UI.  
- Media uploads (group/event images).  

## 14. Change Log
v0.9: Initial comprehensive MVP draft created from product brief.

---
End of Document.

