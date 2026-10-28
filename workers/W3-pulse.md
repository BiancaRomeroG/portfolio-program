# Worker W3 — `pulse`

Include `workers/COMMON.md` first.

- Repo: `D:\Repositorio\pulse` → `github.com/BiancaRomeroG/pulse`
- Spec: `specs/04-pulse.md`
- Phase to execute now: **{{PHASE}}**

## Focus for this repo
- This is the systems-design showcase: delivery guarantees, failure handling and observability. Every guarantee in the spec (outbox, idempotent consumers, retry topics, DLQ, replay, trace propagation) needs a test that proves it and a paragraph in `docs/architecture.md` that explains it.
- Build the `EventBus` port and its three adapters first; run the same contract test suite against all of them. The Kafka adapter is the real one; Redis Streams exists only for the free hosted demo and must be labelled as such in the UI.
- Observability is a deliverable, not a bonus: Grafana dashboards are committed JSON, provisioned automatically; a single trace must span ingest → relay → router → dispatcher. Put a screenshot in the docs.
- Kubernetes and Terraform are documentation-heavy: `docs/kubernetes.md` and `docs/terraform.md` must let a reader reproduce `kind` and LocalStack runs with copy-paste commands.
- Free resources the owner must create for the hosted demo (Neon, Upstash, GitHub OAuth app) go in `docs/BLOCKED.md` with exact steps; local and kind paths never depend on them.
- Email: Mailpit locally; Resend adapter only if the owner later supplies a key; hosted demo shows rendered email previews in the UI otherwise.

## Library guidance
- `kafkajs`, `ioredis` (Redis Streams) or `@upstash/redis` for the REST variant, `@opentelemetry/sdk-node` + auto-instrumentations + `@opentelemetry/exporter-trace-otlp-http`, `pino` + `pino-loki`, `nodemailer`, `web-push`, `socket.io` with Nest gateway, `jsonpath-plus`, `handlebars`, `next-auth` (Auth.js v5), `argon2`, `testcontainers` (Redpanda module), `k6` (binary in CI via `grafana/setup-k6-action`).
- Redpanda image `redpandadata/redpanda`, console `redpandadata/console`, LocalStack `localstack/localstack`, Terraform `hashicorp/aws` provider with LocalStack endpoints.
- Check Context7 for kafkajs, OpenTelemetry JS and Auth.js v5 APIs before coding.
