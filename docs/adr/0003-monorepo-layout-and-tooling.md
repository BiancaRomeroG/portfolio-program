# ADR-0003: Per-repo Turborepo layout and shared tooling

- Status: accepted
- Date: 2026-10-28

## Decision
Every project repo uses this layout. Extra apps are allowed when the spec says so.

```
apps/
  web/            Next.js 16 (App Router, TS, Tailwind v4, Motion)
  api/            NestJS 11 (Swagger at /docs, pino, zod)
packages/
  shared/         zod schemas, DTO types, event envelopes, env schema, money helpers
  config/         eslint flat config, tsconfig bases, prettier config
  ui/             (portfolio only) animated primitives
infra/
  docker/         Dockerfiles
  compose/        docker-compose.yml + seed containers
  helm/ terraform/ (pulse only)
docs/             see PLAN.md section 4
.github/          workflows, templates
turbo.json  pnpm-workspace.yaml  package.json  .nvmrc  .npmrc  commitlint.config.cjs  .husky/
```

Tooling:
- Turborepo tasks: `dev`, `build`, `lint`, `typecheck`, `test`, `test:e2e`, `db:migrate`, `db:seed`.
- ESLint flat config + `typescript-eslint` strict-type-checked + `eslint-plugin-import-x`. Prettier with `prettier-plugin-tailwindcss`.
- Tests: Vitest (unit), Supertest via Nest testing module (API e2e), Playwright (web e2e), fast-check (property tests where the spec demands invariants).
- Commits: Conventional Commits enforced by commitlint + husky `commit-msg`. Pre-push runs `turbo lint typecheck test`.
- Releases: release-please GitHub Action, `CHANGELOG.md` auto-generated, tags `vX.Y.Z`.
- Env: a single zod schema in `packages/shared/src/env.ts`; apps call `loadEnv()` at boot and fail fast.
- Docker: multi-stage images (`deps`, `build`, `runner`), non-root user, `HEALTHCHECK`. `docker compose up` must bring the full stack with seeded data.

## Consequences
A reader who learns one repo can navigate all four. Workers copy the template from `portfolio-program/templates` and adapt names only.
