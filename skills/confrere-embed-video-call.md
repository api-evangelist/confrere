---
name: Embed a Confrere video call
description: >-
  Create a Confrere (Compodium) video room server-side, redirect a user into it
  with a single-use URL, and end the session by invalidating the room or ejecting
  a user.
api: openapi/confrere-video-openapi.yml
generated: '2026-07-18'
method: generated
operations:
  - "POST /api/token"
  - "POST /api/room/{id}/invalidate"
  - "POST /api/room/{id}/{userId}/invalidate"
---

# Embed a Confrere video call

Confrere is a privacy-first, embeddable video-consultation platform. The API is a
small **server-side** surface: you mint a single-use room URL and redirect the
user's browser to it. Never expose your API key in the browser.

## Authentication

Every request requires three headers (see `authentication/confrere-authentication.yml`):

- `Authorization: Bearer $TOKEN`
- `X-Vidicue-Key: <your integration API key>`
- `X-Vidicue-Nonce: <numeric nonce>`

## Steps

1. **Create the room URL** — `POST /api/token`.
   - Body requires `roomId` and `userId`. The `roomId` is namespaced as
     `"<integrationPrefix>:<roomId>"`; the id after the prefix allows only
     `[a-zA-Z0-9_-]`, max 100 chars.
   - Optional: `role` (`guest` | `moderator` | `admin`, default `guest`),
     `roomName`, `useLobby`, `roomLock` (waiting room), `clientOptions`
     (UI/feature toggles), `userData.name`.
   - Response is `{ "url": "..." }`. **The URL expires after one minute and can be
     used only once** — hand it straight to the user's browser as a redirect.

2. **Redirect the participant** to the returned `url`. Confrere runs the
   camera/mic/speaker check, lobby, and call in-browser (iframeable, no download).

3. **End the session** — `POST /api/room/{id}/invalidate` invalidates the room
   key so all signed tokens can no longer join, and immediately destroys the room.

4. **Eject one participant** (optional) — `POST /api/room/{id}/{userId}/invalidate`
   invalidates a single user id and immediately kicks that user.

## Conventions & errors

- Idempotency: none published — creation is keyed by your `roomId`; invalidate
  operations are terminal (see `conventions/confrere-conventions.yml`).
- Errors: `400` = invalid payload/roomId, `401` = bad/missing auth headers
  (see `errors/confrere-problem-types.yml`).
- No pagination or list endpoints exist.
