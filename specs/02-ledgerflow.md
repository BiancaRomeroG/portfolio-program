# Spec 02 — `ledgerflow` (Worker W1)

Payments core of Nimbus Pay: an immutable double-entry ledger, payment intents, two simulated providers with signed webhooks, settlement batches and reconciliation. Zero external keys.

Pitch (README): *"A production-shaped payments core: double-entry ledger with enforced invariants, idempotent APIs, signed webhooks from simulated card and QR providers, and daily batch reconciliation with discrepancy cases. Built to be audited, not just demoed."*

## 1. Scope

In: ledger (accounts, journal entries, postings, balances), payment intents with state machine, two simulated providers (`cardsim`, `qrsim`) as a separate app with their own consoles, inbound webhooks (HMAC, replay protection), transactional outbox (events exposed via SSE and an outbox table to Pulse-compatible consumers), settlement batch import (CSV/JSON), reconciliation engine with discrepancy cases, refunds (partial/full), fees, reports, admin dashboard, merchant checkout demo.

Out: real money, KYC, payouts to banks (modelled as a ledger transfer only), multi-currency FX (fixed table allowed), mobile apps.

## 2. Architecture

```
apps/web         Next.js dashboard + checkout demo (Supabase Auth)
apps/api         NestJS: ledger, payments, providers(client), webhooks, settlement, reconciliation, reports, outbox
apps/providers   NestJS: cardsim + qrsim modules, each with REST API, async webhook emitter, failure injection
packages/shared  zod schemas, Money, event catalogue, state machine definitions
infra/compose    postgres, api, providers, web
```

Flow (card): web checkout → `POST /v1/payment-intents` (Idempotency-Key) → api creates intent `created`, calls `cardsim /authorize` → cardsim responds `processing` and later POSTs a signed webhook `authorization.approved` / `declined` after configurable latency → api verifies HMAC + timestamp + nonce, stores raw webhook, transitions intent, writes journal entry (debit `provider_receivable:cardsim`, credit `merchant_payable:<merchant>`; fee split debit `merchant_payable`, credit `revenue_fees`) → outbox row → relay publishes `payment.captured.v1`.

Settlement: cardsim produces a daily batch (lines: providerRef, amount, fee, status). `POST /v1/settlements/import` (or scheduled pull from providers' `/batches/:date`) → reconciliation job matches lines to captured intents: `matched`, `missing_in_ledger`, `missing_in_provider`, `amount_mismatch`, `duplicate`. Each non-match opens a `DiscrepancyCase` with a suggested action; the UI lets an operator resolve (adjusting entry with reason, or mark false positive). Journal entry `settlement`: debit `cash:settlement:cardsim`, credit `provider_receivable:cardsim`.

## 3. Data model (Prisma)

- `Account(id, code unique, type: ASSET|LIABILITY|REVENUE|EXPENSE|EQUITY, currency, name, ownerType: SYSTEM|MERCHANT|PROVIDER, ownerId?)`
- `JournalEntry(id ULID, occurredAt, kind: PAYMENT_CAPTURE|FEE|REFUND|SETTLEMENT|ADJUSTMENT|PAYOUT, reference, memo, createdBy, hash, prevHash)` — immutable, `hash = sha256(prevHash + canonical json)` (tamper-evident chain).
- `Posting(id, entryId, accountId, direction: DEBIT|CREDIT, amountMinor bigint, currency)`
- `BalanceSnapshot(accountId, asOf, debitMinor, creditMinor, version)` — materialized by a job; live balance = snapshot + delta query.
- `PaymentIntent(id, merchantId, amountMinor, currency, provider, status, providerRef?, idempotencyKey, metadata json, timestamps)`
- `PaymentEvent(id, intentId, fromStatus, toStatus, cause, payload json, at)` — audit trail.
- `Refund(id, intentId, amountMinor, status, providerRef?)`
- `WebhookInbox(id, provider, eventId unique, signatureValid, receivedAt, processedAt?, rawBody, headers json, status: RECEIVED|PROCESSED|REJECTED|DUPLICATE)`
- `Outbox(id, type, payload json, createdAt, publishedAt?, attempts)`
- `SettlementBatch(id, provider, businessDate, lineCount, totalMinor, feeMinor, status, importedAt)`
- `SettlementLine(id, batchId, providerRef, amountMinor, feeMinor, status)`
- `DiscrepancyCase(id, batchId, lineId?, intentId?, kind, amountDeltaMinor, status: OPEN|RESOLVED|FALSE_POSITIVE, resolution?, resolvedBy?, resolvedAt?)`
- `IdempotencyRecord(key, scope, requestHash, responseStatus, responseBody, expiresAt)`
- `Merchant(id, name, feeBps, settlementAccountId)`, `User(id, supabaseSub, email, role: ADMIN|OPERATOR|VIEWER)`

Invariants (enforced in code and tested with fast-check):
1. Every journal entry balances per currency: sum(debits) == sum(credits).
2. Entries are append-only; corrections are new entries of kind `ADJUSTMENT` referencing the original.
3. Balances never computed from floats; `Money` is `{ minor: bigint, currency }`.
4. An intent transition is valid only if in the state machine table; invalid transitions return 409 Problem.
5. A webhook with a seen `eventId` is `DUPLICATE` and has no side effects; a timestamp older than 5 minutes is `REJECTED`.
6. Same `Idempotency-Key` + same body → same response; different body → 422.

DB-level guards: a Postgres trigger that rejects `UPDATE`/`DELETE` on `JournalEntry` and `Posting`; a deferred constraint trigger checking per-entry balance at commit.

## 4. API (NestJS, `/v1`)

- `POST /payment-intents`, `GET /payment-intents/:id`, `GET /payment-intents?merchantId&status&cursor`, `POST /payment-intents/:id/cancel`, `POST /payment-intents/:id/refunds`
- `POST /webhooks/:provider` (public, HMAC), `GET /webhooks/inbox?status`, `POST /webhooks/inbox/:id/replay`
- `GET /ledger/accounts`, `GET /ledger/accounts/:code/balance?asOf`, `GET /ledger/entries?account&from&to&cursor`, `GET /ledger/entries/:id`, `POST /ledger/adjustments` (ADMIN)
- `POST /settlements/import`, `GET /settlements`, `GET /settlements/:id/lines`, `POST /settlements/:id/reconcile`
- `GET /reconciliation/cases?status`, `POST /reconciliation/cases/:id/resolve`
- `GET /reports/daily?date`, `GET /reports/merchant/:id?from&to`
- `GET /events/stream` (SSE of outbox, for demos and for Pulse), `GET /outbox?published=false`
- `GET /health`, `GET /ready`

Providers app (`apps/providers`, port 4100):
- cardsim: `POST /cardsim/authorize`, `POST /cardsim/capture`, `POST /cardsim/refund`, `GET /cardsim/batches/:date`, `POST /cardsim/admin/chaos` (latencyMs, failureRate, duplicateWebhooks, dropWebhooks).
- qrsim: `POST /qrsim/qr` (returns qr payload + simulator URL), `POST /qrsim/simulate/scan/:qrId` (customer pays), `GET /qrsim/batches/:date`, `POST /qrsim/admin/chaos`.
- Both sign webhooks: `X-Sig-Timestamp`, `X-Sig-Nonce`, `X-Signature: v1=hex(hmac_sha256(secret, ts + "." + nonce + "." + body))`; retry with backoff on non-2xx (max 5).

## 5. Web (Next.js)

Pages: `/login` (Supabase), `/` overview (today's volume, capture rate, open cases, balance cards, live event feed from SSE), `/payments` (table + filters; detail drawer with `StateMachineViz`, event audit, raw provider refs, refund action), `/ledger` (accounts tree with balances; entry explorer; entry detail showing postings and hash chain link), `/settlements` (batches; import; reconcile; per-batch result summary), `/reconciliation` (cases board; resolve dialog with adjustment preview), `/webhooks` (inbox with signature status, raw body `JsonViewer`, replay), `/reports`, `/checkout` (public demo: pick merchant, amount, provider; live status; for QR shows the code and a "simulate customer scan" button), `/providers` (chaos console: sliders for latency/failure/duplicates; batch generator).

## 6. Tasks

### Phase 1
- [P1-01] Turborepo scaffold: `apps/web`, `apps/api`, `apps/providers`, `packages/shared`, `packages/config`; pnpm, husky, commitlint, release-please, CI, Dockerfiles, compose (postgres 16, api, providers, web).
- [P1-02] `packages/shared`: `Money`, env schema, CloudEvents envelope + catalogue + fixtures, payment state machine table.
- [P1-03] Prisma schema v1 + migration + deterministic seed (Spec 00 §3).
- [P1-04] NestJS base: config, pino, Problem Details filter, request id, throttler, Swagger + OpenAPI export script + CI staleness check, `/health` `/ready`.
- [P1-05] Supabase Auth guard + `User` upsert; web login page with `@supabase/ssr`.
- [P1-06] Web shell: tokens, top bar, nav, theme toggle, `ui` components (Button, Card, Badge, DataTable, JsonViewer, EmptyState, Toast).
- [P1-07] Docs skeleton, ADR-0001 versions, architecture.md v1 (C4 + card flow sequence), runbook v1.

### Phase 2
- [P1-08] Ledger module: accounts, entries, postings, balance query, hash chain, DB triggers, `ADJUSTMENT`. Property tests for invariants 1–3.
- [P1-09] Idempotency interceptor + storage; tests for invariant 6.
- [P1-10] Payment intents: create, state machine, events audit, cancel. Tests for invariant 4.
- [P1-11] `apps/providers` cardsim: authorize/capture/refund, async signed webhooks, chaos endpoints, batch generator.
- [P1-12] `apps/providers` qrsim: qr create, simulate scan, webhooks, batches.
- [P1-13] Webhook inbox: HMAC verify, timestamp/nonce replay protection, dedupe, processing to intent transitions + journal entries + fees. Tests for invariant 5.
- [P1-14] Outbox + relay + SSE stream; event catalogue emission.
- [P1-15] Refunds (partial/full) with ledger entries and provider call.
- [P1-16] Settlement import (JSON/CSV) + pull from providers; reconciliation engine with the five outcomes; discrepancy cases; resolve flow with adjustment entries.
- [P1-17] Reports endpoints (daily, merchant). API e2e suite with Supertest covering happy paths and chaos scenarios (duplicate webhook, late webhook, amount mismatch).
- [P1-18] data-model.md ERD + invariants; ADRs: money representation, hash chain, idempotency storage, reconciliation matching rules.

### Phase 3
- [P1-19] Overview page with live SSE feed and KPI cards.
- [P1-20] Payments list + detail drawer + refund.
- [P1-21] Ledger explorer (accounts tree, entries, postings, hash chain).
- [P1-22] Settlements + reconciliation board + resolve dialog.
- [P1-23] Webhook inbox + replay.
- [P1-24] Public checkout demo + QR simulate; providers chaos console.
- [P1-25] Playwright: login, create payment via checkout, see it captured, run reconciliation, resolve a case. Screenshots into `docs/demo.md`.

### Phase 4
- [P1-26] Deploy: Supabase (schema via Prisma migrate), Render services `ledgerflow-api` and `ledgerflow-providers` (`render.yaml`), Vercel web. Env docs. Cold-start banner in web.
- [P1-27] README final with GIF (checkout → webhook → ledger → reconciliation), badges, Spanish section. `docs/case-study.md`. Release v1.0.0.

### Phase 5
- [P1-28] k6 load test on `POST /payment-intents` with idempotency; results in docs.
- [P1-29] Chaos run documented: 20% failure + 10% duplicate webhooks → zero ledger drift (prove with a reconciliation report).
- [P1-30] TOTP 2FA for ADMIN; security headers; dependency audit.
- [P1-31] Export journal to CSV via a job; "auditor mode" read-only UI.

## 7. Definition of done (Gate 4)
Public checkout creates a payment, provider webhook lands, ledger entry balances, reconciliation shows it matched; chaos console demonstrates duplicate/late webhooks handled; coverage >= 70% domain; OpenAPI committed; docs complete; GIF in README.
