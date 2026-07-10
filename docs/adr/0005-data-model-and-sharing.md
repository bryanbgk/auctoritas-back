# ADR-0005: Data model for multi-user sharing (tasks vs. finance)

Status: Proposed
Date: 2026-07-09

## Context

`doc.md` calls out two opposite sharing defaults in the same system:

- Task management is **shared** between users on a board.
- Financial management is **separate/private** between users by default,
  with the ability to share a specific view (e.g. expenses for one event)
  with specific other users.

Getting this modeled wrong at the Prisma schema level is expensive to fix
later (risk of accidental data leakage between users' finances), so it's
worth deciding explicitly rather than improvising per-endpoint.

## Decision

- **Tasks domain:** `Board` has many `BoardMember` (user + role), `Board`
  has many `Task`. A task is visible to anyone who is a member of its
  board. No per-task ACL is needed for the MVP — sharing granularity is at
  the board level.
- **Finance domain:** every financial record (`Transaction`, `Loan`,
  `Financing`) has a single `ownerId` (the user who created/imported it)
  and is private to that owner by default.
- **Sharing** is modeled as its own first-class Prisma entity rather than
  bolted onto ownership: a `FinanceShare` (or `SharedSpace`) groups a
  subset of transactions/categories and a list of users granted read
  access to that group. This keeps "share this event's expenses with Alice
  and Bob" as an explicit, auditable, revocable object instead of mutating
  ownership or scattering visibility flags across transactions.
- Enforce isolation with **both** an application-layer check (guards from
  [ADR-0004](0004-authentication-and-authorization.md)) **and** a
  PostgreSQL row-level security policy on finance tables, so a bug in one
  layer doesn't fully compromise privacy on its own.
- Every share grant/revoke is written to an **audit log** table (who
  shared what, with whom, when), satisfying the PRD's auditability
  requirement.

## Alternatives considered

- **Tagging transactions with a `visibility` enum (private/shared) instead
  of a separate sharing entity:** simpler, but doesn't support sharing the
  same transaction with different user subsets for different purposes, and
  makes revocation and auditing harder to reason about.
- **Multi-tenant "organization" model** where finance data is scoped to an
  org instead of a user: rejected — the requirement is explicitly
  per-user-private-by-default with selective sharing, not an org-wide
  shared space; an org model would default to *more* sharing than the
  product needs.

## Consequences

- Requires an extra Prisma model (`FinanceShare` + membership) compared to
  a flag-based approach, but keeps sharing semantics explicit and
  revocable.
- Row-level security policies need to be kept in sync with Prisma
  migrations (Prisma doesn't manage RLS policies directly — they'll need
  raw SQL migrations) — worth covering with an integration test that
  asserts cross-user finance queries return nothing without a share grant.
- Task and finance domains can evolve independently — a future
  "task-linked expense" feature (e.g. cost attached to a task) would need
  its own ADR on how the two sharing models interact.
