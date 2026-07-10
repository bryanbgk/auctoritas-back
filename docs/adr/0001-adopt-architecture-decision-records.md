# ADR-0001: Adopt Architecture Decision Records (ADRs)

Status: Accepted
Date: 2026-07-09

## Context

Auctoritas is developed collaboratively across separate repositories
(`auctoritas-back`, `auctoritas-front`, and later mobile/extension repos),
with technical decisions in each repo that affect the whole system. Without
a formal record, the reasoning behind each decision gets lost over time, and
cross-repo consistency (e.g. auth strategy, realtime transport) is easy to
drift out of sync.

## Decision

Record relevant architectural decisions as ADRs under `docs/adr/` in the
repo where the decision is primarily owned, numbered sequentially, following
the format: Context, Decision, Consequences, and Alternatives considered.
Superseded decisions are marked "Superseded by ADR-XXXX" rather than
removed. Decisions that affect both repos (e.g. the front/back contract,
realtime transport, git workflow) get a mirrored ADR in both repos, with
each pointing at the other via relative links.

## Consequences

- Every non-trivial technical decision should come with an ADR, before or
  shortly after implementation.
- New collaborators can understand the "why" behind past choices by reading
  `docs/adr/` in the relevant repo.
- Cross-repo decisions require updating two files; this is accepted
  overhead in exchange for each repo staying self-contained (see
  [ADR-0003](0003-backend-technology-stack.md) on the no-monorepo decision).
