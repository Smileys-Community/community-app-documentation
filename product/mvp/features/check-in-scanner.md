# Feature Requirements — Check-in & Scanner (MVP)

## Goals
Fast, reliable on-site verification using browser camera.

## User Stories
- As Host/Admin, I scan attendee tickets and mark them as checked-in.

## Acceptance Criteria
- Web scanner (mobile-friendly) requests camera permission; decodes QR; calls check-in API.
- Check-in API validates token, event, and status; idempotent (multiple scans safe).
- UI feedback states: valid → “Checked in”; already checked-in → show timestamp; invalid → error.

## Performance
- p95 < 250ms for check-in endpoint; resilient to burst traffic.

## Audit
- Store checked_in_at and checked_in_by; reversal requires Group Admin.
