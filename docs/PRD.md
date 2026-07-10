# PRD — Auctoritas

Status: Draft (TBD)
Last updated: 2026-07-09
Source: [doc.md](../doc.md)
Repos: `auctoritas-back` (this repo), `auctoritas-front`

## 1. Context and problem

There is currently no single tool that combines (a) collaborative task
management with multiple views and (b) personal/shared finance management
with automated data import (documents and credit card statements).
Auctoritas is being built from scratch, collaboratively, across separate
front/back (and later mobile/extension) repositories, prioritizing
free-to-use technologies (or generous free tiers).

## 2. Goals

- Let multiple users collaborate on shared task boards, with real-time
  updates.
- Let the same tasks be viewed either as a kanban board or as an agenda/
  calendar, organized by each task's due date, split across boards.
- Let each user manage their finances **separately and privately** by
  default, with the option to share specific views with other users (e.g.
  expenses for a shared event).
- Reduce manual entry of financial data via document (PDF/photo) import and
  credit card statement integration (Open Finance).
- Ship a technical foundation that later extends to mobile (Expo) and a
  browser extension (Plasmo/WXT) without a rewrite.

## 3. Non-goals (for now)

- Advanced financial reporting (BI, forecasting) is not an initial goal.
- Multi-currency/i18n support is not an initial goal.
- Mobile app and browser extension are planned platforms but not MVP
  deliverables — see [ADR-0005 (front)](../../auctoritas-front/docs/adr/0005-multi-platform-strategy.md)
  for how the architecture keeps that door open.
- **Explicitly out of scope / parked idea:** a historical-events timeline
  feature (Wikipedia-fed) was raised as an unrelated idea and is not part of
  this product's scope.

## 4. Personas

- **Task collaborator:** participates in one or more shared task boards
  with other users (e.g. the project's own dev team).
- **Individual finance user:** manages their own finances, with data
  isolated from other users unless explicitly shared.
- **User group (shared event/expense):** a subset of users who share a
  specific financial view (e.g. a trip, an event).

## 5. Functional requirements

### 5.1 Users, login, and authorization
- User registration/login, backend-issued JWT (see
  [back ADR-0004](adr/0004-authentication-and-authorization.md)).
- API access authorization per resource (a user can only access task data
  for boards they belong to, and their own financial data, unless
  explicitly shared).
- A permission model that clearly separates the "tasks" scope (shared) from
  the "finance" scope (private by default). See
  [back ADR-0005](adr/0005-data-model-and-sharing.md).

### 5.2 Task management
- Task CRUD (title, description, status, date, assignee, board).
- **Kanban** view (status columns, drag-and-drop).
- **Calendar/agenda** view (same tasks, organized by date).
- Real-time updates between users connected to the same board (parallel
  usage). See [back ADR-0006](adr/0006-realtime-communication.md).

### 5.3 Finance management
- Manual entry of income/expenses.
- Import of financial data from **photos/documents (PDF)**, with automatic
  extraction of relevant data. See
  [back ADR-0007](adr/0007-financial-document-ingestion.md).
- Integration with **credit card** statements/expenses (e.g. Nubank) via
  Open Finance, to import transactions automatically. See
  [back ADR-0008](adr/0008-credit-card-integration.md).
- **Loan/debt** management (amount, installments, rates, due dates).
- Selective sharing of a financial view (e.g. expenses for an event) between
  specific users, without exposing the rest of each user's finances.

### 5.4 Financing
- Registration and tracking of financing arrangements (e.g. property,
  vehicle): total amount, installments, interest, outstanding balance,
  dates.
- Relationship to the loans/debts module (possibly the same domain, see
  section 8).

## 6. Non-functional requirements

- **Real time:** multiple users editing the same board should see updates
  near-instantly, without silently losing data on collisions.
- **Financial privacy:** one user's financial data must never be visible to
  another user without explicit, auditable sharing.
- **Cost:** prioritize free/open-source technologies and free hosting tiers
  in the initial phase (Neon/Supabase, Upstash, Vercel — see
  [back ADR-0003](adr/0003-backend-technology-stack.md)).
- **Front/back decoupling:** front and back live in separate repositories
  and communicate only through a versioned OpenAPI/Swagger contract; no
  shared runtime code or monorepo (see
  [back ADR-0003](adr/0003-backend-technology-stack.md)).
- **Auditability:** sensitive financial operations (import, sharing,
  deletion) must be traceable.

## 7. Stack and tooling (defined in [doc.md](../doc.md))

| Topic | Choice |
|---|---|
| Version control | GitHub, separate repos (front/back/mobile/extension) |
| Database | PostgreSQL (Neon or Supabase, free tier) |
| ORM | Prisma |
| Cache | Redis (Upstash, free tier) |
| Frontend | Next.js |
| Backend | NestJS |
| Front↔back contract | OpenAPI/Swagger (generated by NestJS) + generated TS client (orval) |
| Auth | Own JWT (Passport.js) in the backend |
| Realtime (collaborative kanban) | Supabase Realtime or Socket.io |
| Storage (PDFs, receipts) | Supabase Storage or Cloudflare R2 |
| PDF reading | `pdf-parse`/`pdf.js` (+ Tesseract.js for scanned docs) + LLM for structured extraction |
| Credit card integration (BR) | Pluggy or Belvo (Open Finance) |
| Mobile | Expo (React Native) |
| Browser extension | Plasmo or WXT |
| Team task tracking | Trello |

Details and rationale split across the backend ADRs (this repo,
[docs/adr/](adr/)) and the frontend ADRs (`auctoritas-front/docs/adr/`).

## 8. Open questions

1. Are loans/debts and financing the same data domain with subtypes, or
   separate entities?
2. Pluggy vs. Belvo for Open Finance — needs an evaluation spike (pricing
   past sandbox, bank coverage, DX) before committing; see
   [back ADR-0008](adr/0008-credit-card-integration.md).
3. Supabase Realtime vs. self-hosted Socket.io — needs a decision; see
   [back ADR-0006](adr/0006-realtime-communication.md).
4. What level of granularity should financial sharing have (per
   transaction, per category, per shared "space")?
5. Will there be a single task board per user group, or multiple boards
   with different members?

## 9. Out of scope / next steps

- Define wireframes for kanban/calendar and finance screens.
- Detail entities and data model (Prisma schema).
- Prioritize the backlog in Trello based on sections 5.2–5.4.
- Run the Pluggy/Belvo and Supabase Realtime/Socket.io evaluation spikes
  before committing to timelines that depend on them.
