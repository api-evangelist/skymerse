---
name: Get active NOTAMs for airports
description: Fetch and interpret live NOTAMs for up to five ICAO airports, with optional time-window filtering.
api: openapi/skymerse-notamify-openapi-original.json
operations:
- 'GET /notams'
- 'GET /notams/nearby'
- 'GET /notams/raw'
---

# Get active NOTAMs for airports

Use the Notamify API V2 to retrieve live NOTAMs with AI interpretations.

## Auth
Send `Authorization: Bearer YOUR_API_KEY` on every request. Generate a key at
https://notamify.com/api-manager. All calls are TLS-only. Base URL:
`https://api.notamify.com/api/v2`.

## Steps
1. Call `GET /notams?locations=EPWA` — up to five ICAO codes (comma or repeated
   param). Add `starts_at`/`ends_at` (ISO 8601) to filter by time window.
2. Read the paginated result: `notams[]`, `total_count`, `page`, `per_page`.
   Page through with `page`/`per_page`.
3. Each NOTAM carries `icao_message` (raw) plus `interpretation` — `category`,
   `subcategory`, `affected_elements[]`, and `schedules` in RRULE format for
   client-side time filtering.
4. For proximity use `GET /notams/nearby`; for uninterpreted raw text use
   `GET /notams/raw`.

## Rules
- Rate limit: 20 requests/minute (archive endpoint 10/minute). Usage is metered
  by credits; successful requests deduct credits.
- Errors return `application/json`. Handle 404 (no NOTAMs for parameters).
- Do not expose the API key in client-side code.
