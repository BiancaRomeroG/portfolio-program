# ADR-0006: Authentication strategy per project

- Status: accepted
- Date: 2026-10-28

## Decision
- **ledgerflow** and **incident-copilot**: Supabase Auth (email + password, Google OAuth). The Next.js app uses `@supabase/ssr`. NestJS validates the Supabase JWT with a `SupabaseAuthGuard` (JWKS / JWT secret from env) and maps `sub` to an internal `User` row on first request. Roles stored in the app DB, not in Supabase metadata. No RLS; NestJS is the only writer and uses the service connection.
- **pulse**: two audiences. Tenants integrate machine-to-machine with API keys (`pk_live_...` hashed with argon2 in DB) and HMAC-SHA256 request signatures with timestamp + nonce. Humans use the dashboard through Auth.js v5 with the GitHub provider (free OAuth app). This shows a different auth integration than the other two repos.
- Local development: Supabase Auth points at the free cloud project (no local Supabase stack); Postgres runs in Docker. Documented in each runbook.
- 2FA (TOTP) is a Phase 5 stretch goal in ledgerflow using `otplib`; no SMS anywhere.

## Consequences
- Real OAuth integrations (Google, GitHub) without cost.
- Auth code the owner can explain in interviews: JWT validation, API key hashing, HMAC signatures, replay protection.
