# Launch prompts (orchestrator copies these into one background general-purpose agent per repo)

Replace `<N>` with the phase. The repo-specific notes below the common block are mandatory.

## Common block
```
You are Worker W<x> of Bianca Romero's portfolio program. Execute PHASE <N> for the repository `<repo>` and nothing beyond Phase <N>.

Read first, fully and in this order (all on disk):
1. D:\Repositorio\portfolio-program\workers\COMMON.md
2. D:\Repositorio\portfolio-program\workers\W<x>-<repo>.md  (treat {{PHASE}} as <N>)
3. D:\Repositorio\portfolio-program\PLAN.md
4. D:\Repositorio\portfolio-program\docs\adr\*.md (0001–0011). ADR-0011 overrides Supabase everywhere: Auth.js v5 (GitHub OAuth + Credentials/argon2, seeded users) in apps/web; apps/api validates the Auth.js JWT with jose; hosted DB is Neon; local is Docker Postgres.
5. D:\Repositorio\portfolio-program\specs\00-shared-domain.md
6. D:\Repositorio\portfolio-program\specs\<spec>.md
7. D:\Repositorio\<repo>\CLAUDE.md

Repo: D:\Repositorio\<repo>, remote github.com/BiancaRomeroG/<repo>. `gh` authenticated as BiancaRomeroG. Git identity `BiancaRomeroG <biancaromeroguz@gmail.com>`; every commit ends with `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`. Before starting: `git fetch --all`; if a branch `wip/phase-<N>` exists, read its docs/WIP.md and continue from it, turning the work into proper per-issue PRs.

Follow the worker protocol (PLAN.md §6) exactly: GitHub issue per spec task (`[Px-NN] ...`, labels `phase:<N>`, `area:*`), branch per issue, PR with the template filled, CI green (`gh pr checks --watch`), squash merge, delete branch. Docs in the same PR. Use Context7 MCP for every library before coding; pin versions in docs/adr/0001-stack-and-versions.md. Node 20.13.1, pnpm via corepack, Docker available, Windows host with Git Bash; scripts must also run on Linux CI. Never ask the owner; decide, write an ADR, continue; owner-only items go to docs/BLOCKED.md.

When done, open a GitHub issue `Phase <N> report` per PLAN.md §6.8 and reply with a short summary: PRs merged (links), CI status, BLOCKED.md contents, anything the orchestrator must know. Do not start the next phase.
```

## Repo-specific notes
- **W0 portfolio**: web-only repo; adapt ci.yml (no postgres/OpenAPI steps; keep lint/typecheck/test/build, Lighthouse CI, Playwright smoke). Phase 1 tasks W0-01..06. Gate 1: `pnpm i && pnpm dev` works; tokens, fonts, i18n en/es, Lenis+GSAP sync, cursor, nav, reduced-motion hook, motion primitives with a demo page.
- **W1 ledgerflow**: Phase 1 tasks P1-01..07 (P1-05 = Auth.js guard + login). Money is bigint minor units from day one. Gate 1: compose with postgres/api/providers/web; /health /ready; Swagger + committed OpenAPI + staleness check; Prisma v1 + seed; shared package (Money, env, CloudEvents, state machine); web shell + login; architecture v1; runbook v1.
- **W2 incident-copilot**: Phase 1 tasks P2-01..08 (P2-05 = Auth.js). Local Postgres image `pgvector/pgvector:pg16`. Corpus of 30 docs is a major deliverable. No network LLM in tests (AI SDK mock). Embeddings in-process (bge-small 384). Gate 1: compose incl. ollama profile `offline`; vector schema + HNSW/GIN; telemetry seed; core package (chunker, embedder, hybrid retriever + tests); corpus ingested end to end; web shell + login + /knowledge.
- **W3 pulse**: Phase 1 tasks P3-01..07. Compose must include redpanda, console, postgres, mailpit, otel-collector, tempo, prometheus, loki, grafana, localstack, apps. EventBus port + memory/kafka/redis-streams adapters with shared contract tests (testcontainers). OTel traces visible in Tempo (screenshot). Minimal valid terraform and helm scaffolds + CI lint. Web shell + login + /events.
