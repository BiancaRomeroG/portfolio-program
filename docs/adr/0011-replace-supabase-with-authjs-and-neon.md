# ADR-0011: Replace Supabase (Auth + Postgres) with Auth.js + Neon in all repos

- Status: accepted (supersedes the Supabase parts of ADR-0004 and ADR-0006)
- Date: 2026-10-28

## Context
Phase 0 tried to create the Supabase projects `ledgerflow` and `incident-copilot`. The owner's Supabase organization already has two active free projects (`Familia`, `Koinor`) and the free plan allows two per member. Pausing or deleting the owner's existing projects is not a decision the orchestrator may take. We need a hosted Postgres and an auth story with no card and no owner blocking.

## Decision
1. **Auth** in `ledgerflow`, `incident-copilot` and `pulse` dashboards: **Auth.js v5** (`next-auth`) in `apps/web` with two providers: GitHub OAuth (free OAuth app) and Credentials (email + password hashed with argon2, seeded demo users). Session strategy JWT, signed with `AUTH_SECRET`. `apps/api` validates the Auth.js JWT with a `JwtAuthGuard` (`jose`, same secret, `HS256`) and maps `sub` to the internal `User`. Roles live in the app DB. `pulse` tenant API keys + HMAC stay as in ADR-0006.
2. **Hosted Postgres** for all three: **Neon** free plan, one Neon project `nimbus-pay` with three databases `ledgerflow`, `incident_copilot`, `pulse` (pgvector extension enabled on `incident_copilot`; Neon supports `CREATE EXTENSION vector`). Owner creates the Neon account and pastes the three `DATABASE_URL`s before Phase 4; until then everything runs on Docker Postgres.
3. Supabase MCP remains available; if the owner later frees a slot, a repo may add Supabase Auth as an additional provider through an ADR, but it is no longer required anywhere.
4. Local development has zero external dependencies for auth: Credentials provider with seeded users (`admin@nimbuspay.dev` / password from seed) works offline; GitHub OAuth is optional locally.

## Consequences
- No owner action is on the critical path until Phase 4.
- One consistent auth implementation across the three repos (less to document, easier to explain in interviews: OAuth + credentials + JWT validation in a separate API).
- Demo login works instantly for recruiters with seeded credentials shown on the login page.
- `PLAN.md` section 7 owner tasks: remove Supabase, add Neon (one project, three databases) and one GitHub OAuth app per web app (three callbacks) or a single OAuth app per deployment URL.
- Workers W1 and W2 must replace Supabase references in their specs with this ADR; specs are not rewritten, this ADR takes precedence.
