# Worker W2 — `incident-copilot`

Include `workers/COMMON.md` first.

- Repo: `D:\Repositorio\incident-copilot` → `github.com/BiancaRomeroG/incident-copilot`
- Spec: `specs/03-incident-copilot.md`
- Phase to execute now: **{{PHASE}}**

## Focus for this repo
- The differentiators are: hybrid retrieval you can inspect, an agent whose tool calls are visible, an MCP server that works from Claude Code, and evals that gate CI. Build each so it can be demonstrated in under 30 seconds.
- Corpus quality drives everything. Write the 30 documents as if they were real Nimbus Pay runbooks: concrete commands, thresholds, service names from Spec 00, owners, escalation paths. Postmortems follow a standard template (impact, timeline, root cause, action items). They must be consistent with LedgerFlow and Pulse specs.
- No key required to develop: use Ollama locally if `GROQ_API_KEY` is absent; make the provider switch visible in the UI. For unit and API tests, mock the model with the AI SDK test utilities so CI never calls a network LLM; only the nightly evals job uses Groq (secret `GROQ_API_KEY`, add to BLOCKED.md until the owner sets it with `gh secret set`).
- Embeddings run in-process with transformers.js; cache model files under a Docker volume; warm up on boot; `/ready` reports it.
- Supabase: project `incident-copilot` created by the orchestrator with pgvector; local dev uses Docker Postgres with the `pgvector/pgvector:pg16` image.

## Library guidance
- `ai` (Vercel AI SDK, latest stable), `@ai-sdk/groq`, `ollama-ai-provider`, `@huggingface/transformers`, `@modelcontextprotocol/sdk` (server + client for tests), `pdf-parse` (PDF text), `unified`/`remark` for Markdown heading-aware chunking, `prisma` with raw SQL for `vector` columns, `nestjs-pino`, `supertest`, `fast-check` where useful (RRF properties).
- Check Context7 for current AI SDK and MCP SDK APIs before writing code; both move fast.
