# ADR-0004: Authentication and authorization model

Status: Proposed
Date: 2026-07-09

## Context

`doc.md` specifies "own JWT (Passport.js) in the backend" for auth, and
separately calls out needing to "verify how this works with other
technologies" — i.e. authorization needs to work as a real API access
control layer, not just a login screen, since the API will eventually be
consumed by web, mobile, and a browser extension.

The system also has two scopes with very different sharing rules:

- **Tasks**: shared by design between members of a board.
- **Finance**: private by default per user, with explicit, granular sharing
  to specific other users (see [ADR-0005](0005-data-model-and-sharing.md)).

## Decision

- Implement auth with **Passport.js JWT strategy** inside NestJS, per
  `doc.md`. Issue short-lived **access tokens** (e.g. 15 min) and
  longer-lived **refresh tokens**, rotated on use.
- Passwords hashed with **argon2** (bcrypt as fallback) — never stored or
  logged in plaintext.
- Because clients include a browser extension and (later) mobile, don't
  hard-couple the refresh-token transport to httpOnly cookies: the web app
  can use cookies, but the API should also support returning the refresh
  token in the response body for clients where cookies aren't the natural
  fit (extension/mobile), stored in platform-appropriate secure storage on
  that client.
- Authorization is enforced with **NestJS guards** at the resource level,
  not just the route level:
  - A `BoardMembershipGuard` checks board membership for task-related
    endpoints.
  - A `FinanceOwnershipGuard` checks resource ownership (or an explicit
    share grant, see [ADR-0005](0005-data-model-and-sharing.md)) for
    finance-related endpoints.
- Model roles as **board-scoped** (member/admin of a specific task board)
  rather than global roles, since task collaboration and financial privacy
  are scoped differently per user.
- Leave room for OAuth-based social login (GitHub, Google) later as an
  additional Passport strategy — ship email/password first for the MVP.

## Alternatives considered

- **Session-based auth (server-side sessions in Redis):** viable and
  arguably simpler to revoke, but JWT + refresh was chosen (also matching
  `doc.md`'s explicit call for "own JWT") to keep the API stateless and
  easier to consume from multiple current/future clients (web, extension,
  mobile) without a shared session-store assumption.
- **Third-party auth-as-a-service (Auth0, Clerk, Supabase Auth):** would
  cut implementation time, and Supabase Auth in particular would piggyback
  on Supabase if chosen for DB/Storage/Realtime — but `doc.md` explicitly
  specifies a self-built JWT/Passport solution, and fine-grained
  per-resource authorization would still need app-layer implementation
  regardless. Worth revisiting only if auth becomes a maintenance burden.

## Consequences

- The API must implement refresh-token rotation and revocation
  (e.g. on logout, or treat reuse of an already-rotated refresh token as a
  compromise signal and revoke the whole token family).
- Every finance-related endpoint must check ownership/sharing explicitly
  via guards, not by convention in controllers, so a missed check doesn't
  silently leak financial data.
- Supporting both cookie and body-based refresh-token delivery adds a
  little complexity to the auth module, but avoids a rework when the
  browser extension and mobile clients are built.
