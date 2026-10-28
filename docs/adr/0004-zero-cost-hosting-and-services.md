# ADR-0004: Zero-cost hosting; no credit card; no paid third-party APIs

- Status: accepted
- Date: 2026-10-28

## Context
The owner will not provide a credit card and does not want keys from paid services (Stripe, Twilio, WhatsApp, OpenAI). Demos still need public URLs.

## Decision

| Concern | Choice | Why |
|---------|--------|-----|
| Web hosting | Vercel Hobby | Free, no card, Next.js native, preview deployments per PR. |
| API hosting | Render free Web Service (Docker) | Free, no card. Sleeps after 15 min idle; cold start ~30-60 s. README must state it and the web app shows a "waking up the API" state. |
| Postgres (ledgerflow, incident-copilot) | Supabase free | Free, no card, pgvector available, Auth included. Limit 2 active projects per org. |
| Postgres (pulse) | Neon free | Supabase limit reached; Neon free needs no card. |
| Auth | Supabase Auth / Auth.js GitHub | See ADR-0006. |
| LLM | Groq free tier (default), Ollama local (fallback) | Free keys without card. See ADR-0007. |
| Embeddings | In-process transformers.js | No key at all. |
| Kafka hosted | Not available free without card. Hosted demo uses Upstash Redis Streams through the same `EventBus` port | Ports and adapters; Kafka stays the real transport locally and in kind. See ADR-0008. |
| Email | Mailpit (local). Hosted: Resend free (100/day, no card) optional, otherwise "email preview" page | No Twilio/WhatsApp anywhere. |
| SMS/WhatsApp | Not used | Replaced by Web Push (VAPID self-generated) and in-app WebSocket. |
| Payments | Simulated providers built in-repo | See specs/02-ledgerflow.md. |
| AWS | LocalStack in Docker + Terraform | Proves IaC and AWS APIs with no account. |
| Observability | Grafana OSS stack in Docker; Sentry free optional | |
| Analytics | Vercel Analytics free | |

## Consequences
- Everything runs fully offline with `docker compose up`; hosted demos are a subset wired through adapters.
- If any free tier changes, the swap is one adapter and one ADR.
- Render cold starts are a known UX cost; mitigated with a loading state and a GIF in the README.
