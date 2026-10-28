# Worker W1 — `ledgerflow`

Include `workers/COMMON.md` first.

- Repo: `D:\Repositorio\ledgerflow` → `github.com/BiancaRomeroG/ledgerflow`
- Spec: `specs/02-ledgerflow.md`
- Phase to execute now: **{{PHASE}}**

## Focus for this repo
- Correctness is the product. The six invariants in the spec section 3 must each have a property-based test (fast-check) and, where stated, a DB-level guard. A reviewer should be able to break the ledger only by editing the tests.
- Money: `bigint` minor units end to end (Prisma `BigInt`, JSON serialised as string). No `number` for amounts anywhere outside display formatting.
- Idempotency, HMAC webhook verification, replay protection and the outbox are the interview talking points. Keep them in small, readable modules with TSDoc.
- The simulated providers must feel real: asynchronous webhooks, retries, signature headers, a chaos console. They are what makes the demo credible without Stripe.
- Supabase: the orchestrator created project `ledgerflow`; connection values are in `docs/BLOCKED.md` placeholders until the owner pastes them into `.env`. Local dev uses Docker Postgres regardless.
- Event contracts: copy the catalogue from Spec 00 exactly; Pulse (another repo) will consume these shapes.

## Library guidance
- NestJS 11, `@nestjs/swagger`, `nestjs-zod` (or `zod` pipes), `@nestjs/throttler`, `nestjs-pino`, Prisma, `ulid`, `fast-check`, `supertest`, `@supabase/ssr` + `@supabase/supabase-js` (web), `jose` (JWT verification in the guard).
- SSE via Nest `@Sse()`; keep the outbox relay as a Nest scheduled job (`@nestjs/schedule`) in Phase 2.
