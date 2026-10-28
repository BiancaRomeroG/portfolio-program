# ADR-0008: Eventing transport, delivery guarantees and observability for pulse

- Status: accepted
- Date: 2026-10-28

## Decision
- Broker: **Redpanda** (Kafka API compatible, single container, no ZooKeeper) locally and in kind. Client: `kafkajs`. Redpanda Console included in compose for inspection.
- Port: `EventBus` interface (`publish`, `subscribe`, `ack`, `nack`, `deadLetter`). Adapters: `KafkaEventBus` (real one) and `RedisStreamsEventBus` (Upstash free, used only by the hosted demo). Adapter chosen by `EVENT_BUS` env. Tests run against both using testcontainers (Redpanda) and an in-memory adapter for unit tests.
- Envelope: CloudEvents 1.0 JSON, `type` like `com.nimbuspay.ledger.payment.captured`, `specversion`, `id` (ULID), `source`, `time`, `datacontenttype`, `data`. Schemas in zod, versioned by `type` suffix `.v1`.
- Guarantees: transactional outbox on producers (same DB transaction as the state change; relay publishes and marks sent), consumer idempotency table keyed by event `id`, retry topics with exponential backoff (`*.retry.1m`, `*.retry.5m`, `*.retry.30m`), dead-letter topic `*.dlq` with replay endpoint and UI button.
- Observability: OpenTelemetry SDK for Node in every service; trace context propagated through Kafka headers (`traceparent`); OTel Collector to Tempo (traces), Prometheus (metrics), Loki (logs via pino). Grafana dashboards provisioned from JSON in `infra/grafana/`. One dashboard per service plus a "delivery funnel" dashboard.
- Kubernetes: Helm chart `infra/helm/pulse` with one Deployment per service, HPA on the dispatcher, ConfigMaps/Secrets, kind cluster script `infra/k8s/kind-up.sh`.
- Terraform: `infra/terraform` against LocalStack provisioning S3 (exports and attachments), SQS (external DLQ archive consumed by an exporter job), Secrets Manager (tenant signing secrets). `tflint` and `terraform validate` in CI.

## Consequences
- Local and kind runs are the real architecture; the hosted demo is explicitly labelled "lite transport".
- The repo becomes the owner's reference for "event-driven + k8s + IaC + OTel" questions in interviews.
