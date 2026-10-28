# Spec 03 — `incident-copilot` (Worker W2)

On-call assistant for Nimbus Pay: RAG over runbooks and postmortems with citations, an agent with tools that inspects (seeded) telemetry and drafts incident timelines, exposed both as a web app and as an MCP server for Claude Code and other clients. Automated evals in CI.

Pitch: *"An AI on-call copilot that answers with citations, calls tools against real telemetry, builds incident timelines, and ships with an eval suite that fails CI when quality regresses. Runs on free-tier Groq or fully offline on Ollama."*

## 1. Scope

In: document ingestion (Markdown, PDF, plain text) with chunking and metadata; hybrid retrieval (pgvector + full-text, RRF, optional rerank); chat with streaming, citations and tool-call visualization; agent tools (`searchRunbooks`, `getServiceHealth`, `queryLogs`, `getRecentDeploys`, `openIncident`, `appendTimeline`, `proposeRemediation`); incident objects with timelines; knowledge base management UI; MCP server (stdio + Streamable HTTP) exposing the same tools and resources; eval suite (retrieval + answer quality) with CI gate; seeded telemetry simulator (services, logs, deploys, metrics) matching Spec 00 services.

Out: real integrations with Sentry/Datadog (adapters stubbed behind interfaces), fine-tuning, voice.

## 2. Architecture

```
apps/web        Next.js: chat, incidents, knowledge base, evals dashboard (Supabase Auth)
apps/api        NestJS: ingest, retrieval, chat (AI SDK streaming), agent tools, incidents, telemetry sim, evals runner
apps/mcp        MCP server (TypeScript SDK) wrapping the same tool implementations via packages/core; stdio locally, Streamable HTTP hosted
packages/core   domain: chunking, embeddings (transformers.js), retrieval, tools, prompts, eval metrics (pure TS, no Nest)
packages/shared zod schemas, event catalogue, env
corpus/         30+ Markdown docs: runbooks, postmortems, architecture notes about LedgerFlow and Pulse, on-call policies; plus 3 PDFs
infra/compose   postgres(pgvector), api, web, mcp, ollama (profile: offline)
```

Request flow (chat): web → `POST /v1/chat` (AI SDK `streamText` with tools, `maxSteps` 6) → model decides tools → tool results appended → final answer with inline citation markers `[1]` mapped to chunk ids → web renders a citations panel with highlighted chunk text and source link.

Retrieval: query → embed (bge-small) → pgvector top-40 cosine + tsvector top-40 (`websearch_to_tsquery`) → RRF → top-8 → optional rerank → context budget by tokens → prompt with system rules (cite or say you don't know; never invent commands).

## 3. Data model

- `Document(id, title, source, kind: RUNBOOK|POSTMORTEM|ARCH|POLICY|UPLOAD, service?, tags[], contentHash, createdAt, indexedAt?, status)`
- `Chunk(id, documentId, ordinal, text, tokens, embedding vector(384), tsv tsvector, headingPath, metadata json)` with HNSW index and GIN index.
- `Conversation(id, userId, title, createdAt)`, `Message(id, conversationId, role, content, toolCalls json, citations json, usage json, latencyMs, model)`
- `Incident(id, title, severity: SEV1..SEV4, service, status: OPEN|MITIGATED|RESOLVED, openedAt, resolvedAt?, summary?)`, `TimelineEntry(id, incidentId, at, kind: NOTE|TOOL|ACTION|STATUS, text, actor)`
- Telemetry sim: `Service(id, name, owner)`, `Deploy(id, serviceId, version, at, author, status)`, `LogLine(id, serviceId, at, level, message, traceId?)`, `MetricPoint(serviceId, name, at, value)` seeded for 7 days including 3 scripted incidents (provider webhook backlog, DLQ growth in Pulse, ledger reconciliation mismatch spike).
- `EvalRun(id, at, model, dataset, metrics json, passed)`, `EvalCase(id, runId, question, expected, answer, retrieved json, scores json)`
- `User(id, supabaseSub, email, role)`

## 4. API

- `POST /v1/documents` (multipart upload or JSON), `POST /v1/documents/:id/reindex`, `GET /v1/documents`, `DELETE /v1/documents/:id`, `POST /v1/ingest/corpus` (loads `corpus/`)
- `POST /v1/search` (debug: ranked chunks with scores from each retriever and the fused rank)
- `POST /v1/chat` (AI SDK data stream), `GET /v1/conversations`, `GET /v1/conversations/:id`
- `GET /v1/incidents`, `POST /v1/incidents`, `GET /v1/incidents/:id`, `POST /v1/incidents/:id/timeline`, `POST /v1/incidents/:id/status`
- Telemetry sim: `GET /v1/telemetry/services`, `GET /v1/telemetry/services/:id/health`, `GET /v1/telemetry/logs?service&level&from&to&q`, `GET /v1/telemetry/deploys?service`, `POST /v1/telemetry/scenarios/:name/trigger` (replays a scripted incident live for demos)
- `POST /v1/evals/run`, `GET /v1/evals/runs`, `GET /v1/evals/runs/:id`
- `/health`, `/ready` (includes "embedding model loaded")

Tools (shared by API agent and MCP):

| tool | input | output |
|------|-------|--------|
| searchRunbooks | query, service?, k | chunks with citations |
| getServiceHealth | service | status, error rate, p95, last deploy |
| queryLogs | service, level?, q?, window | matching log lines |
| getRecentDeploys | service, hours | deploys |
| openIncident | title, severity, service | incident |
| appendTimeline | incidentId, text, kind | entry |
| proposeRemediation | incidentId | structured plan with cited runbook steps (LLM, must cite) |

MCP (`apps/mcp`): tools above; resources `runbook://{documentId}`, `incident://{id}`; prompts `triage-incident`, `write-postmortem`. Document how to register in Claude Code (`.mcp.json` snippet) and in Claude Desktop.

## 5. Evals (`packages/core/evals` + `apps/api` runner)

Dataset `evals/dataset.jsonl` with 40 cases: question, expected answer key points, expected source document ids, optional required tool. Metrics:
- Retrieval: recall@8 of expected documents, MRR.
- Answer: faithfulness and relevance via LLM-judge (Groq) with a strict rubric returning 0–1; citation coverage (every claim sentence has a citation).
- Tool use: expected tool called when required.
Thresholds: recall@8 >= 0.85, faithfulness >= 0.8, citation coverage >= 0.9. CI job `evals-nightly.yml` runs on schedule and on PRs labelled `evals`; posts a markdown summary as a job summary and stores the run in DB when `DATABASE_URL` is set. Fails below thresholds.

## 6. Web

`/login`; `/` chat (conversation list, streaming answer, tool-call pills expanding to input/output, citations side panel with chunk highlight, provider selector when both configured, rate-limit retry state); `/incidents` (board by severity; detail with timeline, "ask copilot about this incident" button pre-filled, status changes); `/knowledge` (documents table, upload, reindex, chunk preview, search debugger showing both retrievers and fused rank); `/telemetry` (services grid with sparklines, log viewer, scenario trigger buttons for the demo); `/evals` (runs list, per-case table with scores, trend chart).

## 7. Tasks

### Phase 1
- [P2-01] Turborepo scaffold: `apps/web`, `apps/api`, `apps/mcp`, `packages/core`, `packages/shared`, `packages/config`; tooling, CI, Dockerfiles, compose (postgres with pgvector, api, web, mcp; `ollama` under profile `offline`).
- [P2-02] Prisma schema v1 with `vector(384)` (raw SQL migration for extension + HNSW/GIN indexes); seed telemetry (7 days, 3 scenarios) and users.
- [P2-03] `packages/core`: chunker (heading-aware, ~400 tokens, 15% overlap), embedder (transformers.js, lazy load, batch), `Retriever` interface with pgvector + fulltext + RRF.
- [P2-04] NestJS base (pino, Problem Details, request id, throttler, Swagger + export + staleness check, health/ready with model warm-up).
- [P2-05] Supabase Auth guard + web login.
- [P2-06] Corpus v1: 30 Markdown docs about LedgerFlow and Pulse (runbooks for webhook backlog, DLQ growth, reconciliation mismatch, Render cold start, Supabase connection limits, Kafka consumer lag; 5 postmortems; architecture notes; on-call policy) + 3 PDFs generated from Markdown. `POST /v1/ingest/corpus` end to end.
- [P2-07] Web shell + ui components; `/knowledge` list working against the API.
- [P2-08] Docs skeleton, ADR-0001 versions, architecture.md v1, runbook v1 (Groq key, Ollama option).

### Phase 2
- [P2-09] AI provider factory (`groq` | `ollama`), retry/backoff, usage logging.
- [P2-10] `POST /v1/search` debug endpoint; retrieval unit tests with a fixed mini-corpus (recall assertions).
- [P2-11] Chat endpoint with AI SDK streaming, system prompt with citation rules, context budgeting, conversation persistence.
- [P2-12] Tools in `packages/core/tools` with zod schemas; telemetry sim queries; incident tools.
- [P2-13] Agent loop (`maxSteps`, tool results streamed), citations post-processor mapping `[n]` to chunk ids, guard against uncited claims (re-ask once).
- [P2-14] Incidents module + timeline + status transitions.
- [P2-15] `apps/mcp`: server with tools, resources, prompts; stdio entry + Streamable HTTP; integration test using the MCP client SDK; docs for Claude Code registration.
- [P2-16] Evals package: dataset v1 (40 cases), metrics, judge prompt, runner, CLI `pnpm evals`, nightly workflow with thresholds and job summary.
- [P2-17] Optional rerank with cross-encoder; ADR on whether it stays (measure with evals).
- [P2-18] API e2e tests (ingest → search → chat with a mocked model via AI SDK mock provider); data-model.md; ADRs: chunking strategy, hybrid retrieval, judge rubric, MCP transport.

### Phase 3
- [P2-19] Chat UI with streaming, tool pills, citations panel, provider selector, rate-limit state.
- [P2-20] Incidents board + detail + "ask copilot".
- [P2-21] Knowledge UI: upload, reindex, chunk preview, search debugger.
- [P2-22] Telemetry UI with sparklines and scenario triggers.
- [P2-23] Evals dashboard.
- [P2-24] Playwright: login, ask a question, see citation, trigger scenario, open incident from chat, run evals (mocked). Screenshots to `docs/demo.md`.

### Phase 4
- [P2-25] Deploy: Supabase (pgvector enabled), Render `incident-copilot-api` (model cache warm-up) and `incident-copilot-mcp` (HTTP), Vercel web. Owner provides `GROQ_API_KEY` (BLOCKED.md if missing; demo falls back to a recorded-transcript mode).
- [P2-26] README with GIF (question → tools → cited answer → incident timeline), MCP usage section with `.mcp.json`, eval badge, Spanish section; `docs/case-study.md`; v1.0.0.

### Phase 5
- [P2-27] Eval dataset v2 (80 cases) with adversarial questions; publish eval history chart in README.
- [P2-28] Streaming tool-call UI polish; read-only conversation sharing links.
- [P2-29] Postmortem generator prompt producing Markdown from an incident timeline; export to `corpus/` and auto-reindex (closing the loop).
- [P2-30] Observability: token/latency metrics per model; cost estimate panel (even if free).

## 8. Definition of done (Gate 4)
Hosted chat answers a seeded question with correct citations and at least one tool call; MCP server usable from Claude Code following the README; nightly evals green with thresholds; coverage >= 70% on `packages/core`; docs complete; GIF in README.
