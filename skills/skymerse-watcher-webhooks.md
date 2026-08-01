---
name: Subscribe to NOTAM changes via the Watcher API
description: Create a Watcher listener that pushes matching interpreted NOTAMs to a signed webhook and verify signatures.
api: openapi/skymerse-notamify-openapi-original.json
operations:
- 'GET /notams'
---

# Subscribe to NOTAM changes via the Watcher API

The Watcher API delivers matching NOTAM interpretations to a webhook (or up to 3
emails) as they publish and change.

## Steps
1. Create a listener (see docs: Creating Listeners) with a `webhook_url`
   (external URLs only) and `applies_to` filters — aircraft class/designator/
   family, traffic/operation/phase scope, time windows (dates, days, UTC hours),
   and optional QCode patterns.
2. Receive `POST` deliveries with `kind: interpretation` (a matching NOTAM) or
   `kind: lifecycle` (a previously delivered NOTAM was cancelled `NOTAMC` or
   replaced `NOTAMR`). Payload: `listener_id`, `kind`, `event_id`, `notam`,
   `change`, `context`, `sent_at`.
3. Deduplicate on `event_id`.
4. Verify the `X-Notamify-Signature` header: parse `t=` and each `v1=`, compute
   `HMAC_SHA256(secret, "<timestamp>.<raw_body>")`, compare to each `v1`, and
   enforce a 10-minute timestamp tolerance. During secret rotation two `v1`
   signatures arrive for a 3-hour grace window. The `notamify-sdk`
   `verify_signature()` helper does this for you.

## Rules
- Deliveries pause after 25 consecutive failures — return 2xx quickly.
- A sandbox environment is available for testing listeners.
