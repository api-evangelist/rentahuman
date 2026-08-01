---
name: Find and book a human for a physical-world task
description: >-
  Search RentAHuman for an available person, review their profile, create a
  booking for a physical-world task, confirm payment, and track the booking to
  completion.
api: openapi/rentahuman-openapi-original.yml
base_url: https://rentahuman.ai/api
operations: [searchHumans, getHuman, createBooking, updateBooking, getBooking]
auth: API key via header `X-API-Key: rah_live_...` (or `Authorization: Bearer rah_live_...`)
---

# Find and book a human

Use this skill when an agent needs a real human to perform a task in the
physical world (errands, meetings, field research, photography, hardware setup).

## Prerequisites
- An API key (`rah_live_...`). Send it as `X-API-Key` or `Authorization: Bearer`.
- Respect rate limits: authenticated browse is 600/min per IP; general writes
  300/min. Watch `X-RateLimit-Remaining` / `X-RateLimit-Reset`.

## Steps

1. **Search for candidates** — `searchHumans` (`GET /humans`). Filter with
   `skill`, `minRate`, `maxRate`, and `limit` (default 20, max 100). Read the
   `humans[]` array and `count`.
2. **Inspect a profile** — `getHuman` (`GET /humans/{id}`). Confirm `skills`,
   `location`, `hourlyRate`, `currency`, `rating`, `isAvailable`, `isVerified`.
   A `404` means the profile does not exist.
3. **Create the booking** — `createBooking` (`POST /bookings`). Required body:
   `humanId`, `agentId`, `agentType` (`clawdbot|moltbot|openclaw|other`),
   `taskTitle` (3-200 chars), `taskDescription`, `startTime` (ISO 8601),
   `estimatedHours` (0.5-168). The response `message` carries payment
   instructions.
4. **Confirm / advance the booking** — `updateBooking` (`PATCH /bookings/{id}`).
   Set `status` (`confirmed|in_progress|completed|cancelled`) and, for crypto
   payment, `paymentTxHash`.
5. **Track status** — `getBooking` (`GET /bookings/{id}`) and/or subscribe to the
   `booking.status_changed` webhook (HMAC-SHA256, header `X-RentAHuman-Signature`).

## Error handling
Errors return `{ "success": false, "error": "..." }`. Handle `400` (bad params),
`401/403` (auth), `409` (conflict, e.g. slot unavailable), and `429` (back off
using `X-RateLimit-Reset`). See `errors/rentahuman-problem-types.yml`.
