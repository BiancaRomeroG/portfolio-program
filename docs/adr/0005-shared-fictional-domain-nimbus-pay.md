# ADR-0005: One fictional company across the three projects

- Status: accepted
- Date: 2026-10-28

## Context
Three unrelated demos read like tutorials. Three systems of the same company read like experience.

## Decision
All projects model **Nimbus Pay**, a fictional LATAM payments company (see `specs/00-shared-domain.md`).
- `ledgerflow` is its payments core: ledger, payment intents, simulated providers, settlement and reconciliation.
- `pulse` is its notification platform: receives domain events (including LedgerFlow's) and delivers email, push, in-app and outbound webhooks.
- `incident-copilot` is its on-call assistant: runbooks and postmortems about LedgerFlow and Pulse, with tools that read their (seeded) telemetry.

Integration is by contract, not by runtime dependency: each repo ships seed data and a simulator for the others' events. Event envelopes follow CloudEvents 1.0 and are duplicated in each repo's `packages/shared/events`.

## Consequences
- The portfolio tells one story: "I built the money core, the eventing layer and the AI on-call tooling of a payments company."
- Each repo still runs alone. No cross-repo imports.
