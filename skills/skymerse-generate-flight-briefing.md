---
name: Generate an AI flight briefing
description: Submit an async flight briefing job over a route and poll for the completed operational briefing.
api: openapi/skymerse-notamify-openapi-original.json
operations:
- 'POST /notams/briefing'
- 'GET /notams/briefing/{uuid}'
- 'GET /notams/briefing/simple'
---

# Generate an AI flight briefing

Produce a route briefing that prioritizes the NOTAMs that matter for a specific
flight.

## Auth
`Authorization: Bearer YOUR_API_KEY`. Base URL `https://api.notamify.com/api/v2`.

## Steps
1. `POST /notams/briefing` with a `GenerateFlightBriefingRequest` body:
   `locations[]`, `origin_runway`, `destination_runway`,
   `destination_procedure`, `aircraft_type`, `aircraft_details`. A `201`
   returns the briefing job. A `400` means an invalid request body.
2. Poll `GET /notams/briefing/{uuid}` for job status. `202` = still processing,
   `200` = complete (`status`, `response`), `404` = job not found, `500` = job
   failed. Back off between polls.
3. The completed briefing returns `critical_operational_restrictions[]` and a
   human-readable `text` with an interactive chip system.
4. For a quick single-airport summary without a job, call
   `GET /notams/briefing/simple` (returns `location`, `summary`, `notam_ids`).

## Rules
- Briefing endpoints consume credits; poll, do not busy-loop.
- 20 requests/minute rate limit.
- To rank NOTAMs for a route without a full briefing, use
  `POST /notams/prioritisation`.
