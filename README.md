# Portfolio Program — Bianca Romero

Repositorio de coordinación. Aquí vive el plan maestro, las decisiones (ADRs), las especificaciones de cada proyecto, los templates de documentación y los briefs de cada worker (agente).

**Nada se construye aquí.** El código vive en 4 repos independientes:

| # | Repo | Qué es | Worker |
|---|------|--------|--------|
| 0 | `portfolio` | Sitio portafolio (Next.js, concepto visual "Event Stream") | W0 |
| 1 | `ledgerflow` | Ledger de doble partida + pasarelas de pago simuladas + conciliación | W1 |
| 2 | `incident-copilot` | Copiloto de incidentes: RAG + agente con tools + servidor MCP + evals | W2 |
| 3 | `pulse` | Plataforma de notificaciones event-driven (Kafka, CloudEvents, outbox, DLQ, k8s, OTel) | W3 |

Los tres proyectos modelan la misma empresa ficticia, **Nimbus Pay** (ver `specs/00-shared-domain.md`): LedgerFlow mueve el dinero, Pulse avisa lo que pasa, Incident Copilot ayuda cuando algo se rompe. Esa narrativa es lo que une el portafolio.

## Cómo leer este repo

1. `PLAN.md` — fases, gates, protocolo de workers, definición de terminado.
2. `docs/adr/` — decisiones tomadas el 2026-10-28. Cualquier worker que necesite desviarse escribe un ADR nuevo en su repo, no pregunta.
3. `specs/` — una spec por proyecto: alcance, arquitectura, modelo de datos, API, UI, tareas por fase con IDs.
4. `templates/` — archivos que cada repo copia tal cual en Fase 1 (README, PR template, ADR template, CLAUDE.md, CI).
5. `workers/` — el prompt exacto con el que se lanza cada worker.

## Restricciones no negociables

- Cero servicios de pago. Cero tarjeta de crédito. Keys solo de free tiers sin tarjeta (Groq, Supabase, Neon, Vercel, Render, Upstash, GitHub OAuth).
- Todo Node/TypeScript. Nada de .NET, Java, Python.
- Inglés en código, commits, docs. README con sección en español.
- Cada cambio entra por PR con template lleno, CI verde, squash merge. Aunque el worker trabaje solo.
- Docs se actualizan en el mismo PR que el código. PR sin docs = PR incompleto.

## Estado

Ver `PLAN.md#estado` (el orquestador lo actualiza al cerrar cada gate).
