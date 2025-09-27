# Feature Requirements — Notifications (Email, MVP)

## Goals
Transactional emails to ensure attendance and trust.

## Templates
- Verify email
- RSVP confirmation (with ticket link)
- Event reminder (24h before; optional +2h)
- Event cancelled (if applicable)

## Acceptance Criteria
- Emails sent on events with templating (title, date/time, map link, “View ticket”).
- Verified sending domain (DKIM, SPF, DMARC); bounce/complaint handling.
- Opt-out for marketing (none in MVP); transactional emails always sent.

## Metrics
- Delivery rate, open rate (where allowed), bounce and complaint rates.
