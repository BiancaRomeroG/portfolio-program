# Worker W0 — `portfolio`

Include `workers/COMMON.md` first.

- Repo: `D:\Repositorio\portfolio` → `github.com/BiancaRomeroG/portfolio`
- Spec: `specs/01-portfolio-site.md`
- Phase to execute now: **{{PHASE}}**

## Focus for this repo
- This is the public face. Visual quality and motion polish matter as much as code quality. Compare yourself against award-level personal sites (Awwwards "portfolio" category) and aim for that bar with the "Event Stream" concept, not a template look.
- Performance and accessibility thresholds are hard limits: Lighthouse performance >= 90, accessibility >= 95; every animation respects `prefers-reduced-motion`.
- Content sources: `D:\Curriculum Vitae\BiancaRomeroG_CV_v2.md` (profile, experience, skills). Keep `[N]` placeholders out of the UI: use qualitative phrasing until the owner supplies numbers, and list the missing numbers in `docs/BLOCKED.md`.
- Project cards reference the three repos. Until each repo reaches Gate 4, show status `in progress` and link to the GitHub repo. Links and names come from `content/projects.json`:
  - `ledgerflow` — "Double-entry ledger, simulated card/QR providers, signed webhooks, batch reconciliation" — NestJS · Next.js · Prisma · Postgres · Supabase Auth
  - `incident-copilot` — "RAG on-call copilot with tools, MCP server and CI evals" — NestJS · Next.js · Vercel AI SDK · pgvector · MCP · Groq/Ollama
  - `pulse` — "Kafka-backed notification platform: outbox, retries, DLQ replay, OpenTelemetry, Helm, Terraform" — NestJS · Next.js · Redpanda · OTel · Kubernetes · Terraform
- Define `docs/case-study-format.md` in Phase 4 so W1–W3 know exactly what to deliver (frontmatter fields, sections, image sizes).
- Owner placeholders you must keep configurable in `content/profile.json`: LinkedIn URL, domain, photo. Default LinkedIn to `https://www.linkedin.com/in/bianca-romero-guzman` and flag it in BLOCKED.md for confirmation.

## Library guidance
- `motion` (import from `motion/react`), `lenis` (`lenis/react`, `ReactLenis root`, sync with GSAP ticker as in Lenis docs), `gsap` + `ScrollTrigger` (GSAP is free including plugins), `next-intl`, `@next/mdx`, `rehype-mermaid` or pre-rendered SVG, `@vercel/analytics`, `@lhci/cli`.
- Canvas work in plain TypeScript inside a `useEffect`, with a pure, unit-tested scheduler for particles; no WebGL.
- Fonts via `next/font/google` (Instrument Serif) and `geist` package (Geist Sans/Mono).
