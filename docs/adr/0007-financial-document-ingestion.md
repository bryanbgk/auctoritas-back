# ADR-0007: Financial document ingestion (PDF/photo import)

Status: Proposed
Date: 2026-07-09

## Context

`doc.md` specifies the tools for this already: `pdf-parse`/`pdf.js` (+
Tesseract.js for scanned documents) plus an LLM for structured extraction,
with storage in Supabase Storage or Cloudflare R2. This ADR fixes how those
pieces fit together and settles the storage choice.

## Decision

- Process document imports **asynchronously** via a job queue (BullMQ on
  the Redis/Upstash instance from
  [ADR-0003](0003-backend-technology-stack.md)): the upload endpoint stores
  the file and enqueues a job; the client is notified over the WebSocket
  gateway ([ADR-0006](0006-realtime-communication.md)) when extraction
  completes.
- **Extraction pipeline:**
  1. `pdf-parse`/`pdf.js` first, for PDFs that already have a text layer
     (the common case for bank/card statements) — no OCR needed.
  2. **Tesseract.js** (self-hosted OCR) for scanned PDFs and photos where a
     text layer isn't available.
  3. An **LLM extraction step** on the resulting raw text to pull out
     structured fields (amount, date, merchant/category), since regex-only
     parsing struggles with the variety of statement/receipt formats. Use
     a cheap/fast model for this (e.g. Haiku-class) given the task is
     narrow extraction, not open-ended reasoning.
- Present the result to the user as a **pre-filled, editable draft
  transaction** rather than auto-committing it — both OCR and LLM
  extraction are expected to be imperfect, and silently trusting them
  risks corrupting financial records.
- **Storage:** use whichever of Supabase Storage / Cloudflare R2 matches
  the Postgres hosting decision in
  [ADR-0003](0003-backend-technology-stack.md) — if Supabase is chosen for
  Postgres, use Supabase Storage to keep one vendor/dashboard; if Neon is
  chosen for Postgres, use Cloudflare R2 (Neon has no storage product).
  Either way, files are stored in object storage, never on the API's local
  disk, so the API stays stateless and horizontally scalable.
- Every import job's raw file, extracted text, and final parsed values are
  retained (at least temporarily) so a bad extraction can be traced and
  corrected, satisfying the PRD's auditability requirement.

## Alternatives considered

- **Synchronous extraction in the request/response cycle:** rejected — OCR
  and LLM calls on photos can take several seconds, a bad fit for a
  blocking HTTP request.
- **Fully automatic import with no user review step:** rejected — given
  OCR/LLM error rates, silently trusting extracted financial figures risks
  incorrect balances with no easy way for the user to notice.
- **Managed OCR API (Google Vision, AWS Textract) instead of Tesseract.js:**
  `doc.md` specifies Tesseract.js; kept as a fallback/upgrade path only if
  self-hosted OCR proves inaccurate in practice, since managed OCR free
  tiers are limited and this would add a paid dependency.

## Consequences

- Requires a background worker process (or a worker mode within the NestJS
  app) in addition to the API itself, plus an LLM API budget (even a cheap
  model has marginal cost per document — track usage).
- Extraction accuracy will vary by document quality; the review-before-
  commit UX step is load-bearing for data correctness, not optional
  polish.
- Storage vendor choice is coupled to the Postgres hosting decision — don't
  decide this ADR's storage question independently of
  [ADR-0003](0003-backend-technology-stack.md).
