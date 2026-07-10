# ADR-0008: Credit card / bank integration strategy

Status: Proposed
Date: 2026-07-09

## Context

`doc.md` names **Pluggy or Belvo** — both licensed Brazilian Open Finance
aggregators — as the intended way to "integrate with possible credit cards
to import expenses (Nubank?)". Using a licensed aggregator (rather than
screen-scraping or a direct bank integration) is already the right call:
it avoids storing user bank credentials and offloads regulatory compliance
to the aggregator. What's not yet decided is **which one**.

## Decision

- Integrate via a **licensed Open Finance aggregator** — Pluggy or Belvo —
  never build direct bank scraping or store end-user bank credentials.
- **Do not pick one yet without a spike.** Both are viable; run a short
  evaluation before committing, comparing:
  - Sandbox/free-tier limits and pricing past sandbox.
  - Coverage of the specific institutions users actually need (starting
    with Nubank, per `doc.md`).
  - SDK/DX quality for a NestJS backend, and webhook reliability for
    transaction sync.
  - Data residency/compliance posture, since this touches financial data
    subject to LGPD.
- Until the spike is done and one is chosen, **MVP ships with manual/file-
  based import** (CSV/OFX/PDF via the same pipeline as
  [ADR-0007](0007-financial-document-ingestion.md)) so the finance module
  isn't blocked on the aggregator decision. OFX in particular is a
  structured format most Brazilian banks/cards (including Nubank) already
  export, needing only a parser — no OCR.
- Design the `Transaction` model (see
  [ADR-0005](0005-data-model-and-sharing.md)) with a `source` field
  (`manual` | `document_import` | `csv_import` | `open_finance`) from the
  start, so wiring up Pluggy/Belvo later is additive, not a migration.

## Alternatives considered

- **Screen-scraping bank/card sites:** rejected — fragile against UI
  changes, likely against banks' terms of service, and requires storing
  user banking credentials, a disproportionate security liability. `doc.md`
  already steers away from this by naming Pluggy/Belvo specifically.
- **Building directly against Nubank's or each bank's own Open Finance
  API:** rejected — becoming a registered Open Finance participant per
  bank is a much heavier compliance path than going through an aggregator
  that's already licensed for this.
- **Deferring all card integration indefinitely, manual entry only:**
  rejected as the end state — it defeats a core stated goal of reducing
  manual entry — but accepted as the *interim* MVP state while the
  Pluggy/Belvo spike happens.

## Consequences

- The finance module ships on schedule with manual/CSV/OFX/PDF import,
  without waiting on an aggregator contract/integration.
- The `source` field on transactions avoids a data migration if/when the
  live aggregator integration is added later.
- Someone needs to own the Pluggy vs. Belvo spike as an explicit backlog
  item (see PRD section 8, item 2) — without it, this ADR stays half-decided
  indefinitely.
