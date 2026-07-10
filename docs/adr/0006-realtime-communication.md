# ADR-0006: Realtime communication for collaborative boards

Status: Proposed
Date: 2026-07-09

## Context

`doc.md` requires "parallel use and real-time updates" for the task
management module, and lists two candidate technologies without deciding
between them: **Supabase Realtime** or **Socket.io**. This ADR makes that
call. It's mirrored in
[`auctoritas-front` ADR-0004](../../../auctoritas-front/docs/adr/0004-realtime-client-integration.md)
since both repos need to agree on the transport.

## Decision

Use **NestJS's WebSocket gateway** (`@nestjs/websockets`, on Socket.io)
with the **Redis adapter** (`@socket.io/redis-adapter`), rather than
Supabase Realtime, for the MVP:

- The backend already needs Redis (Upstash) for caching and the document-
  ingestion job queue ([ADR-0007](0007-financial-document-ingestion.md)),
  so the Socket.io+Redis adapter reuses infrastructure already in the
  stack instead of adding a new one.
- Authorization for realtime events (a user may only join `board:<id>`
  rooms for boards they're a member of) is easiest to enforce in the same
  NestJS auth/guard layer already built for the REST API
  ([ADR-0004](0004-authentication-and-authorization.md)), rather than
  configuring a second, separate authorization model in Supabase Realtime
  (which authorizes via Postgres RLS policies on replicated tables — a
  different mental model from the REST guards).
- Supabase Realtime is Postgres-change-driven (listens to table changes),
  which is a good fit if the DB itself is the single source of truth for
  every consumer, but Auctoritas's realtime need is closer to
  "broadcast this board update to connected clients," which a
  general-purpose WebSocket gateway handles directly without going through
  a DB write-then-listen round trip.

Clients join a room per board (`board:<id>`) after authenticating the
socket connection with the same JWT used for the REST API. Real-time is
**task/board-only** for the MVP; finance data does not get live multi-user
sync initially (private by default, rarely edited concurrently) —
revisit if shared finance spaces (ADR-0005) see concurrent editing in
practice.

Concurrent edits to the same task use **last-write-wins** for the MVP, with
updates carrying an `updatedAt` timestamp so clients can detect (not
silently resolve) stale overwrites later if this proves insufficient.

## Alternatives considered

- **Supabase Realtime:** rejected as the primary transport per the
  reasoning above (auth model mismatch with the REST guards), but stays
  attractive **if** Supabase ends up hosting Postgres/Storage too
  ([ADR-0003](0003-backend-technology-stack.md)), since it would remove a
  moving part. Revisit if Supabase is chosen as the DB/storage provider and
  the RLS-based authorization model proves sufficient.
- **Polling (short-interval REST fetches):** simplest to implement, but
  wastes requests and adds visible latency to a feature whose entire value
  is feeling instantaneous. Kept as a reasonable fallback for clients where
  WebSockets are blocked (e.g. restrictive corporate networks).
- **Server-Sent Events (SSE):** simpler than WebSockets for one-directional
  updates, but the board will likely need client-to-server presence/
  typing-style signals eventually, so a bidirectional channel is preferred.

## Consequences

- The API process must run with the Redis adapter if deployed across
  multiple instances (already planned, not incremental cost).
- `auctoritas-front` needs a Socket.io client wired into the board/calendar
  views, and a reconnect/resync strategy (re-fetch board state on
  reconnect to cover missed events) — see
  [front ADR-0004](../../../auctoritas-front/docs/adr/0004-realtime-client-integration.md).
- If Supabase is later chosen for Postgres, this decision should be
  revisited rather than assumed settled, since it changes the tradeoff.
