# ADR-0002: Git workflow and branching strategy

Status: Accepted
Date: 2026-07-09

## Context

`doc.md`'s "Definições técnicas" section already sketches a workflow:
branch from `DEVELOP`, develop, open a PR back to `DEVELOP`, using a
`features/#0001-Task-name` branch naming pattern and Conventional Commits
prefixed with an issue number. This ADR formalizes it so both repos apply
it consistently.

## Decision

- **Trunk-ish flow with a `develop` integration branch:** `main` reflects
  what's deployed; `develop` is the integration branch feature branches
  target.
  1. Branch from `develop`.
  2. Implement the change.
  3. Open a PR into `develop`.
  4. Merge `develop` into `main` on release.
- **Branch naming:** `<type>/#<issue-number>-<short-description>`, e.g.
  `features/#0001-kanban-board`, `fix/#0042-refresh-token-rotation`.
  `<issue-number>` refers to the Trello card ID/number.
- **Commit messages:** Conventional Commits
  (https://www.conventionalcommits.org/en/v1.0.0/), prefixed with the issue
  number, e.g. `#0001 feat: add kanban drag-and-drop`.
- Applies identically in `auctoritas-back` and `auctoritas-front` (and
  future mobile/extension repos) so history and tooling (changelogs,
  commit linting) stay consistent across the split repos.

## Alternatives considered

- **Trunk-based development (direct to `main` with feature flags):**
  simpler, but the team explicitly wants a `develop` staging branch per
  `doc.md`; revisit once release cadence and CI maturity justify the
  switch.
- **GitFlow with separate `release/*` and `hotfix/*` branches:** more
  ceremony than a small, early-stage team needs right now; the lighter
  `develop`→`main` flow can graduate into full GitFlow later if release
  management gets more complex.

## Consequences

- CI should target PRs into `develop` (lint/test/build) and a separate,
  stricter check (or manual approval) for `develop`→`main`.
- Commit-lint (e.g. commitlint + husky) can be added to enforce the
  Conventional Commits format automatically.
- Trello card numbers become a de facto cross-referencing key between
  commits, branches, and cards — worth keeping stable once cards are
  created.
