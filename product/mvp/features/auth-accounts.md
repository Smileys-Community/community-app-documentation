# Feature Requirements — Auth & Accounts (MVP)

## Goals
Secure, low-friction signup/login with email and social providers; verified emails required to RSVP.

## User Stories
- As a user, I sign up/in with email or Google/Facebook.
- As a user, I can reset my password and verify my email.

## Scope & Rules
- Email is unique identifier; social login resolves to existing account by verified email.
- Email verification required before RSVP.

## Acceptance Criteria
- Signup/login endpoints with rate limiting and bot defenses.
- Email verification link (time-boxed); verified status stored.
- Password reset flow with time-boxed token.
- Sessions: short-lived access token + rotating refresh token; device logout.

## Non-Functional
- p95 < 400ms; lockout after N failed attempts; audit login events.
