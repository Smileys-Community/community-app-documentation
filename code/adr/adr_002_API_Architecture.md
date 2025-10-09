# ADR 002: API Architecture & OpenAPI Specification

## Status
Proposed (Draft) – To be reviewed and iterated during implementation.

## Context
The MVP requires a clear, versioned, secure, and extensible HTTP API to support core user journeys: event discovery, group membership, RSVP lifecycle, ticket generation & verification, and administrative operations. This ADR defines the baseline API architectural decisions and delivers the first comprehensive OpenAPI 3.1 specification to:

- Align backend & frontend contracts early
- Enable parallel development
- Provide a single source of truth for validation, mocks, client generation, and documentation
- Bake in security, consistency, error semantics, and observability hooks

## Architectural Decisions

1. Style: RESTful resource-oriented endpoints with action subpaths (e.g. `/publish`, `/verify`).
2. Versioning: Path-based (`/api` implicit MVP v1). Future breaking change introduces `/api/v2`.
3. Authentication: JWT access token (Bearer) + refresh token (HttpOnly cookie). Some endpoints are public.
4. Authorization: Role-based via server-side claims + resource ownership checks.
5. Consistency: Standard envelope for errors only. Success responses return resource JSON directly (no `data` wrapper) to keep payloads lean.
6. Idempotency: Ticket verification & check-in endpoints are idempotent; repeat operations return stable status objects.
7. Pagination: Cursor-based for listings likely to grow (future), offset-based (page, pageSize) for MVP simplicity. Responses include `page`, `pageSize`, `total`, `items`.
8. Filtering & Sorting: Query parameters with whitelisted names (`sort`, `filter[...]`).
9. Validation: OpenAPI schemas + server runtime validation (e.g. Zod / JOI / class-validator) + database constraints.
10. Observability: Each request must propagate `X-Request-ID` (client may send; server generates if absent) and returns it in response headers.
11. Rate Limiting: Per-IP and per-authenticated user as specified (not expressed in OpenAPI beyond description).
12. Stability: Non-breaking additive changes only after spec publication until tagged release.

## Non-Goals (MVP)
- GraphQL / gRPC interfaces
- Webhooks (except ESP inbound webhooks handled out-of-band later)
- Bulk mutation endpoints
- Real-time streaming (WebSockets) – possible future for live metrics

## OpenAPI Specification
Below is the OpenAPI 3.1 compliant YAML. Keep this file authoritative; generate downstream artifacts (Postman collection, SDKs) from it.

```yaml
openapi: 3.1.0
info:
  title: Community Events Platform API
  version: 1.0.0-draft
  description: |
    REST API for Community Events MVP. Provides authentication, groups, events, RSVPs,
    ticketing, check-in, and administrative capabilities.
    
    Security:
      - Public endpoints require no auth.
      - Authenticated endpoints require Bearer access token.
      - Refresh handled via cookie-based refresh token.
  contact:
    name: Platform Team
    email: api@community.local
  license:
    name: MIT
servers:
  - url: https://api.dev.community.local/api
    description: Development
  - url: https://api.staging.community.app/api
    description: Staging
  - url: https://api.community.app/api
    description: Production
tags:
  - name: Auth
  - name: Users
  - name: Groups
  - name: Events
  - name: RSVPs
  - name: Tickets
  - name: Admin
  - name: System
security:
  - bearerAuth: []
paths:
  /auth/register:
    post:
      tags: [Auth]
      summary: Register with email & password
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/RegisterRequest' }
      responses:
        '201':
          description: User created (verification email sent)
          headers:
            X-Request-ID: { description: Correlation ID, schema: { type: string } }
          content:
            application/json:
              schema: { $ref: '#/components/schemas/User' }
        '400': { $ref: '#/components/responses/BadRequest' }
        '409': { $ref: '#/components/responses/Conflict' }
  /auth/login:
    post:
      tags: [Auth]
      summary: Login with email & password
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/LoginRequest' }
      responses:
        '200':
          description: Authenticated
          headers:
            Set-Cookie: { description: HttpOnly refresh token cookie, schema: { type: string } }
          content:
            application/json:
              schema: { $ref: '#/components/schemas/AuthTokens' }
        '401': { $ref: '#/components/responses/Unauthorized' }
  /auth/logout:
    post:
      tags: [Auth]
      summary: Invalidate refresh token & logout
      responses:
        '204': { description: Logged out }
  /auth/refresh:
    post:
      tags: [Auth]
      summary: Refresh access token using refresh cookie
      responses:
        '200':
          description: New access token
          content:
            application/json:
              schema: { $ref: '#/components/schemas/AuthTokens' }
        '401': { $ref: '#/components/responses/Unauthorized' }
  /auth/verify-email:
    post:
      tags: [Auth]
      summary: Verify email with token
      requestBody:
        required: true
        content:
          application/json:
            schema: { type: object, required: [token], properties: { token: { type: string } } }
      responses:
        '200': { description: Email verified, content: { application/json: { schema: { $ref: '#/components/schemas/User' } } } }
        '400': { $ref: '#/components/responses/BadRequest' }
        '410': { description: Verification token expired }
  /auth/resend-verification:
    post:
      tags: [Auth]
      summary: Resend email verification
      responses:
        '202': { description: Verification email resent if eligible }
  /auth/forgot-password:
    post:
      tags: [Auth]
      summary: Request password reset email
      requestBody:
        required: true
        content:
          application/json:
            schema: { type: object, required: [email], properties: { email: { type: string, format: email } } }
      responses:
        '202': { description: Email sent if account exists }
  /auth/reset-password:
    post:
      tags: [Auth]
      summary: Reset password using token
      requestBody:
        required: true
        content:
          application/json:
            schema: { type: object, required: [token,newPassword], properties: { token: { type: string }, newPassword: { type: string, minLength: 8 } } }
      responses:
        '200': { description: Password updated }
        '400': { $ref: '#/components/responses/BadRequest' }
  /users/me:
    get:
      tags: [Users]
      summary: Get current user profile
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/User' } } } }
        '401': { $ref: '#/components/responses/Unauthorized' }
    patch:
      tags: [Users]
      summary: Update current user
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/UpdateUserRequest' }
      responses:
        '200': { description: Updated, content: { application/json: { schema: { $ref: '#/components/schemas/User' } } } }
    delete:
      tags: [Users]
      summary: Delete current account
      responses:
        '202': { description: Deletion scheduled / performed }
  /users/me/groups:
    get:
      tags: [Users]
      summary: List groups current user has joined
      responses:
        '200': { description: OK, content: { application/json: { schema: { type: array, items: { $ref: '#/components/schemas/Group' } } } } }
  /users/me/events:
    get:
      tags: [Users]
      summary: List events (upcoming & past) for current user
      parameters:
        - in: query
          name: status
          schema: { type: string, enum: [upcoming,past,all], default: upcoming }
      responses:
        '200': { description: OK, content: { application/json: { schema: { type: array, items: { $ref: '#/components/schemas/Event' } } } } }
  /users/me/export-data:
    post:
      tags: [Users]
      summary: Request export of user data (GDPR)
      responses:
        '202': { description: Export queued }
  /groups:
    get:
      tags: [Groups]
      summary: List public groups
      parameters:
        - { in: query, name: page, schema: { type: integer, minimum: 1, default: 1 } }
        - { in: query, name: pageSize, schema: { type: integer, minimum: 1, maximum: 100, default: 20 } }
        - { in: query, name: sort, schema: { type: string, enum: [newest,members,active], default: newest } }
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/PaginatedGroups' } } } }
    post:
      tags: [Groups]
      summary: Create group (superadmin only)
      requestBody:
        required: true
        content:
          application/json: { schema: { $ref: '#/components/schemas/CreateGroupRequest' } }
      responses:
        '201': { description: Created, content: { application/json: { schema: { $ref: '#/components/schemas/Group' } } } }
        '403': { $ref: '#/components/responses/Forbidden' }
  /groups/{groupId}:
    get:
      tags: [Groups]
      summary: Get group by ID
      parameters:
        - { in: path, name: groupId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/Group' } } } }
        '404': { $ref: '#/components/responses/NotFound' }
    patch:
      tags: [Groups]
      summary: Update group (superadmin or group admin)
      parameters:
        - { in: path, name: groupId, required: true, schema: { type: string, format: uuid } }
      requestBody:
        required: true
        content:
          application/json: { schema: { $ref: '#/components/schemas/UpdateGroupRequest' } }
      responses:
        '200': { description: Updated, content: { application/json: { schema: { $ref: '#/components/schemas/Group' } } } }
    delete:
      tags: [Groups]
      summary: Delete (deactivate) group (superadmin)
      parameters:
        - { in: path, name: groupId, required: true, schema: { type: string, format: uuid } }
      responses:
        '204': { description: Deleted }
  /groups/{groupId}/join:
    post:
      tags: [Groups]
      summary: Join group
      parameters:
        - { in: path, name: groupId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: Joined, content: { application/json: { schema: { $ref: '#/components/schemas/GroupMembership' } } } }
        '409': { $ref: '#/components/responses/Conflict' }
  /groups/{groupId}/leave:
    post:
      tags: [Groups]
      summary: Leave group
      parameters:
        - { in: path, name: groupId, required: true, schema: { type: string, format: uuid } }
      responses:
        '204': { description: Left }
        '400': { $ref: '#/components/responses/BadRequest' }
  /groups/{groupId}/members:
    get:
      tags: [Groups]
      summary: List group members (admin only)
      parameters:
        - { in: path, name: groupId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: OK, content: { application/json: { schema: { type: array, items: { $ref: '#/components/schemas/GroupMembership' } } } } }
  /groups/{groupId}/admins:
    post:
      tags: [Groups]
      summary: Assign group admin (superadmin)
      parameters:
        - { in: path, name: groupId, required: true, schema: { type: string, format: uuid } }
      requestBody:
        required: true
        content:
          application/json: { schema: { type: object, required: [userId], properties: { userId: { type: string, format: uuid } } } }
      responses:
        '201': { description: Assigned }
    delete:
      tags: [Groups]
      summary: Remove group admin (superadmin)
      parameters:
        - { in: path, name: groupId, required: true, schema: { type: string, format: uuid } }
        - { in: path, name: userId, required: true, schema: { type: string, format: uuid } }
      responses:
        '204': { description: Removed }
  /events:
    get:
      tags: [Events]
      summary: Public events listing
      parameters:
        - { in: query, name: page, schema: { type: integer, minimum: 1, default: 1 } }
        - { in: query, name: pageSize, schema: { type: integer, minimum: 1, maximum: 100, default: 20 } }
        - { in: query, name: dateFrom, schema: { type: string, format: date-time } }
        - { in: query, name: dateTo, schema: { type: string, format: date-time } }
        - { in: query, name: category, schema: { type: string } }
        - { in: query, name: groupId, schema: { type: string, format: uuid } }
        - { in: query, name: sort, schema: { type: string, enum: [date,recent,most_rsvps], default: date } }
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/PaginatedEvents' } } } }
    post:
      tags: [Events]
      summary: Create event (group admin)
      requestBody:
        required: true
        content:
          application/json: { schema: { $ref: '#/components/schemas/CreateEventRequest' } }
      responses:
        '201': { description: Created, content: { application/json: { schema: { $ref: '#/components/schemas/Event' } } } }
  /events/{eventId}:
    get:
      tags: [Events]
      summary: Get event by ID
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/Event' } } } }
        '404': { $ref: '#/components/responses/NotFound' }
    patch:
      tags: [Events]
      summary: Update event (group admin)
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
      requestBody:
        required: true
        content:
          application/json: { schema: { $ref: '#/components/schemas/UpdateEventRequest' } }
      responses:
        '200': { description: Updated, content: { application/json: { schema: { $ref: '#/components/schemas/Event' } } } }
    delete:
      tags: [Events]
      summary: Delete (archive) event (if zero RSVPs)
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
      responses:
        '204': { description: Deleted }
  /events/{eventId}/publish:
    post:
      tags: [Events]
      summary: Publish a draft event
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: Published, content: { application/json: { schema: { $ref: '#/components/schemas/Event' } } } }
  /events/{eventId}/attendees:
    get:
      tags: [Events]
      summary: List RSVP attendees (admin/host)
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: OK, content: { application/json: { schema: { type: array, items: { $ref: '#/components/schemas/RSVP' } } } } }
  /events/{eventId}/hosts:
    post:
      tags: [Events]
      summary: Assign event host
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
      requestBody:
        required: true
        content:
          application/json: { schema: { type: object, required: [userId], properties: { userId: { type: string, format: uuid } } } }
      responses:
        '201': { description: Host assigned }
    delete:
      tags: [Events]
      summary: Remove event host
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
        - { in: path, name: userId, required: true, schema: { type: string, format: uuid } }
      responses:
        '204': { description: Removed }
  /events/{eventId}/rsvp:
    post:
      tags: [RSVPs]
      summary: Create RSVP to event
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
      responses:
        '201': { description: RSVP created, content: { application/json: { schema: { $ref: '#/components/schemas/RSVP' } } } }
        '400': { $ref: '#/components/responses/BadRequest' }
        '409': { $ref: '#/components/responses/Conflict' }
    delete:
      tags: [RSVPs]
      summary: Cancel RSVP
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
      responses:
        '204': { description: Cancelled }
  /rsvps/{ticketId}:
    get:
      tags: [RSVPs]
      summary: Get RSVP/ticket by ticket ID (ticket display)
      parameters:
        - { in: path, name: ticketId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/RSVP' } } } }
        '404': { $ref: '#/components/responses/NotFound' }
  /tickets/{ticketId}/verify:
    post:
      tags: [Tickets]
      summary: Verify ticket status (idempotent)
      parameters:
        - { in: path, name: ticketId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: Verification result, content: { application/json: { schema: { $ref: '#/components/schemas/TicketVerification' } } } }
        '404': { $ref: '#/components/responses/NotFound' }
  /tickets/{ticketId}/checkin:
    post:
      tags: [Tickets]
      summary: Perform check-in (idempotent)
      parameters:
        - { in: path, name: ticketId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: Check-in result, content: { application/json: { schema: { $ref: '#/components/schemas/TicketCheckInResult' } } } }
        '404': { $ref: '#/components/responses/NotFound' }
  /events/{eventId}/checkin-status:
    get:
      tags: [Tickets]
      summary: Check-in dashboard snapshot
      parameters:
        - { in: path, name: eventId, required: true, schema: { type: string, format: uuid } }
      responses:
        '200': { description: Snapshot, content: { application/json: { schema: { $ref: '#/components/schemas/CheckInStatus' } } } }
  /admin/dashboard:
    get:
      tags: [Admin]
      summary: Superadmin dashboard metrics
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/AdminDashboard' } } } }
        '403': { $ref: '#/components/responses/Forbidden' }
  /admin/users:
    get:
      tags: [Admin]
      summary: List users (searchable)
      parameters:
        - { in: query, name: q, schema: { type: string } }
        - { in: query, name: page, schema: { type: integer, default: 1 } }
        - { in: query, name: pageSize, schema: { type: integer, default: 20 } }
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/PaginatedUsers' } } } }
  /admin/audit-logs:
    get:
      tags: [Admin]
      summary: List audit logs
      parameters:
        - { in: query, name: page, schema: { type: integer, default: 1 } }
        - { in: query, name: pageSize, schema: { type: integer, default: 50 } }
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/PaginatedAuditLogs' } } } }
  /admin/analytics:
    get:
      tags: [Admin]
      summary: Platform analytics summary
      responses:
        '200': { description: OK, content: { application/json: { schema: { $ref: '#/components/schemas/AnalyticsSummary' } } } }
  /health:
    get:
      tags: [System]
      summary: Health probe
      responses:
        '200': { description: Healthy, content: { application/json: { schema: { type: object, properties: { status: { type: string }, uptime: { type: number } } } } } }
  /legal/terms:
    get:
      tags: [System]
      summary: Terms of Service
      responses:
        '200': { description: OK, content: { text/markdown: { schema: { type: string } } } }
  /legal/privacy:
    get:
      tags: [System]
      summary: Privacy Policy
      responses:
        '200': { description: OK, content: { text/markdown: { schema: { type: string } } } }
  /legal/cookies:
    get:
      tags: [System]
      summary: Cookie Policy
      responses:
        '200': { description: OK, content: { text/markdown: { schema: { type: string } } } }
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  responses:
    BadRequest:
      description: Invalid input
      content:
        application/json:
          schema: { $ref: '#/components/schemas/Error' }
    Unauthorized:
      description: Authentication required / invalid
      content:
        application/json:
          schema: { $ref: '#/components/schemas/Error' }
    Forbidden:
      description: Insufficient permissions
      content:
        application/json:
          schema: { $ref: '#/components/schemas/Error' }
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema: { $ref: '#/components/schemas/Error' }
    Conflict:
      description: Conflict (duplicate, invalid state)
      content:
        application/json:
          schema: { $ref: '#/components/schemas/Error' }
  schemas:
    Error:
      type: object
      required: [error, message]
      properties:
        error: { type: string, description: Machine error code }
        message: { type: string, description: Human readable message }
        details: { type: object, additionalProperties: true }
        requestId: { type: string }
      example:
        error: VALIDATION_ERROR
        message: Email is invalid
        details: { field: email }
        requestId: 01HF8Q3J6K2YP
    AuthTokens:
      type: object
      properties:
        accessToken: { type: string }
        expiresIn: { type: integer, description: Seconds until access token expiry }
        tokenType: { type: string, enum: [Bearer], default: Bearer }
      required: [accessToken, expiresIn, tokenType]
    RegisterRequest:
      type: object
      required: [email, password, name]
      properties:
        email: { type: string, format: email }
        password: { type: string, minLength: 8 }
        name: { type: string, minLength: 1, maxLength: 100 }
    LoginRequest:
      type: object
      required: [email, password]
      properties:
        email: { type: string, format: email }
        password: { type: string }
    User:
      type: object
      properties:
        id: { type: string, format: uuid }
        email: { type: string, format: email }
        name: { type: string }
        profilePhotoUrl: { type: string, nullable: true }
        emailVerified: { type: boolean }
        role: { type: string, enum: [guest,member,superadmin] }
        createdAt: { type: string, format: date-time }
        updatedAt: { type: string, format: date-time }
      required: [id,email,name,emailVerified,role,createdAt,updatedAt]
    UpdateUserRequest:
      type: object
      properties:
        name: { type: string }
        email: { type: string, format: email }
        bio: { type: string, maxLength: 500 }
        profilePhotoUrl: { type: string }
    Group:
      type: object
      properties:
        id: { type: string, format: uuid }
        name: { type: string }
        slug: { type: string }
        description: { type: string }
        coverImageUrl: { type: string, nullable: true }
        category: { type: string }
        status: { type: string, enum: [active,inactive] }
        memberCount: { type: integer }
        upcomingEventCount: { type: integer }
        createdAt: { type: string, format: date-time }
      required: [id,name,slug,status,memberCount,upcomingEventCount,createdAt]
    CreateGroupRequest:
      type: object
      required: [name, category]
      properties:
        name: { type: string, minLength: 3, maxLength: 100 }
        description: { type: string, maxLength: 2000 }
        category: { type: string }
        coverImageUrl: { type: string }
    UpdateGroupRequest:
      type: object
      properties:
        name: { type: string }
        description: { type: string }
        category: { type: string }
        status: { type: string, enum: [active,inactive] }
        coverImageUrl: { type: string }
    GroupMembership:
      type: object
      properties:
        id: { type: string, format: uuid }
        userId: { type: string, format: uuid }
        groupId: { type: string, format: uuid }
        role: { type: string, enum: [member,admin] }
        joinedAt: { type: string, format: date-time }
      required: [id,userId,groupId,role,joinedAt]
    Event:
      type: object
      properties:
        id: { type: string, format: uuid }
        groupId: { type: string, format: uuid }
        title: { type: string }
        description: { type: string }
        coverImageUrl: { type: string, nullable: true }
        startTime: { type: string, format: date-time }
        endTime: { type: string, format: date-time }
        venueName: { type: string }
        venueAddress: { type: string }
        venueMapUrl: { type: string, nullable: true }
        capacity: { type: integer, nullable: true }
        status: { type: string, enum: [draft,published,cancelled,completed] }
        tags: { type: array, items: { type: string } }
        rsvpCount: { type: integer }
        createdAt: { type: string, format: date-time }
        updatedAt: { type: string, format: date-time }
        publishedAt: { type: string, format: date-time, nullable: true }
      required: [id,groupId,title,startTime,endTime,venueName,venueAddress,status,createdAt,updatedAt]
    CreateEventRequest:
      type: object
      required: [groupId,title,description,startTime,endTime,venueName,venueAddress]
      properties:
        groupId: { type: string, format: uuid }
        title: { type: string, minLength: 5, maxLength: 200 }
        description: { type: string, maxLength: 5000 }
        startTime: { type: string, format: date-time }
        endTime: { type: string, format: date-time }
        venueName: { type: string }
        venueAddress: { type: string }
        venueMapUrl: { type: string }
        capacity: { type: integer, minimum: 1 }
        tags: { type: array, items: { type: string } }
        coverImageUrl: { type: string }
    UpdateEventRequest:
      type: object
      properties:
        title: { type: string }
        description: { type: string }
        startTime: { type: string, format: date-time }
        endTime: { type: string, format: date-time }
        venueName: { type: string }
        venueAddress: { type: string }
        venueMapUrl: { type: string }
        capacity: { type: integer }
        tags: { type: array, items: { type: string } }
        coverImageUrl: { type: string }
        status: { type: string, enum: [draft,published,cancelled,completed] }
    RSVP:
      type: object
      properties:
        id: { type: string, format: uuid }
        eventId: { type: string, format: uuid }
        userId: { type: string, format: uuid }
        ticketId: { type: string, format: uuid }
        status: { type: string, enum: [confirmed,cancelled] }
        checkedInAt: { type: string, format: date-time, nullable: true }
        checkedInBy: { type: string, format: uuid, nullable: true }
        createdAt: { type: string, format: date-time }
        cancelledAt: { type: string, format: date-time, nullable: true }
      required: [id,eventId,userId,ticketId,status,createdAt]
    TicketVerification:
      type: object
      properties:
        ticketId: { type: string, format: uuid }
        eventId: { type: string, format: uuid }
        status: { type: string, enum: [valid,already_checked_in,invalid,wrong_event,cancelled] }
        attendeeName: { type: string, nullable: true }
        checkedInAt: { type: string, format: date-time, nullable: true }
        message: { type: string }
      required: [ticketId,status]
    TicketCheckInResult:
      allOf:
        - { $ref: '#/components/schemas/TicketVerification' }
      properties:
        # status will reflect final state (valid or already_checked_in)
        result: { type: string, enum: [checked_in,already_checked_in] }
      required: [result]
    CheckInStatus:
      type: object
      properties:
        eventId: { type: string, format: uuid }
        totalRsvps: { type: integer }
        checkedIn: { type: integer }
        percentage: { type: number }
        recent: { type: array, items: { type: object, properties: { ticketId: { type: string, format: uuid }, name: { type: string }, checkedInAt: { type: string, format: date-time } }, required: [ticketId,name,checkedInAt] } }
      required: [eventId,totalRsvps,checkedIn,percentage,recent]
    AuditLog:
      type: object
      properties:
        id: { type: string, format: uuid }
        userId: { type: string, format: uuid }
        action: { type: string }
        entityType: { type: string }
        entityId: { type: string, format: uuid }
        changes: { type: object }
        ipAddress: { type: string }
        userAgent: { type: string }
        createdAt: { type: string, format: date-time }
      required: [id,action,entityType,entityId,createdAt]
    AdminDashboard:
      type: object
      properties:
        totalUsers: { type: integer }
        totalGroups: { type: integer }
        totalEvents: { type: integer }
        totalRsvps: { type: integer }
        recentActivity: { type: array, items: { $ref: '#/components/schemas/AuditLog' } }
      required: [totalUsers,totalGroups,totalEvents,totalRsvps,recentActivity]
    AnalyticsSummary:
      type: object
      properties:
        signupConversion: { type: number }
        rsvpConversion: { type: number }
        attendanceRate: { type: number }
        weeklyActiveUsers: { type: integer }
        eventCounts: { type: object, properties: { draft: { type: integer }, published: { type: integer }, completed: { type: integer }, cancelled: { type: integer } } }
      required: [signupConversion,rsvpConversion,attendanceRate,weeklyActiveUsers,eventCounts]
    PaginatedMeta:
      type: object
      properties:
        page: { type: integer }
        pageSize: { type: integer }
        total: { type: integer }
      required: [page,pageSize,total]
    PaginatedGroups:
      allOf:
        - { $ref: '#/components/schemas/PaginatedMeta' }
        - type: object
          properties:
            items: { type: array, items: { $ref: '#/components/schemas/Group' } }
          required: [items]
    PaginatedEvents:
      allOf:
        - { $ref: '#/components/schemas/PaginatedMeta' }
        - type: object
          properties:
            items: { type: array, items: { $ref: '#/components/schemas/Event' } }
          required: [items]
    PaginatedUsers:
      allOf:
        - { $ref: '#/components/schemas/PaginatedMeta' }
        - type: object
          properties:
            items: { type: array, items: { $ref: '#/components/schemas/User' } }
          required: [items]
    PaginatedAuditLogs:
      allOf:
        - { $ref: '#/components/schemas/PaginatedMeta' }
        - type: object
          properties:
            items: { type: array, items: { $ref: '#/components/schemas/AuditLog' } }
          required: [items]
```

## Future Extensions
- Introduce `webhooks` tag for ESP bounce & complaint processing.
- Add OAuth social login callback examples (omitted due to provider redirects complexity; described in separate integration doc).
- Expand error codes enumeration for client resiliency.
- Add ETag/Last-Modified headers for cacheable GET endpoints.
- Introduce cursor pagination for large RSVP lists.

## Tooling Recommendations
- Use `spectral` for linting (`error` objects, naming conventions).
- Generate TypeScript types via `openapi-typescript`.
- Publish spec at `/api/openapi.yaml` for client consumption & Swagger UI.

## Decision Rationale
This structure balances clarity (resource nouns) with pragmatic action endpoints where needed (publish, verify, checkin). It keeps complexity low while remaining extensible for monetization, advanced analytics, and real-time features.

## Status Tracking
Any changes require PR + version bump in `info.version`. Backwards-compatible additions (new endpoints, optional fields) increment patch; breaking changes require new base path version.

---
End of ADR 002.
