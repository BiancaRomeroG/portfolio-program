# ADR-0002: TypeScript end to end, no .NET / Java / Python

- Status: accepted
- Date: 2026-10-28

## Context
The owner's CV already proves .NET, Spring Boot and Laravel through employment. The gap versus 2026 LATAM remote job posts is Node/TypeScript across the stack, modern React, and AI integration in TS. The owner explicitly asked for "all Node/TS with new technologies, nothing .NET".

## Decision
- Runtime: Node 20 LTS (>= 20.13, matches the dev machine). Upgrade to Node 22 LTS when the machine upgrades; record in each repo's ADR-0001.
- Language: TypeScript 5.x, `strict: true`, `noUncheckedIndexedAccess: true`, ESM modules.
- Package manager: pnpm 9 through corepack (`corepack enable && corepack prepare pnpm@latest-9 --activate`).
- Backend: NestJS 11 (most requested Node framework in LATAM postings), Prisma ORM, zod validation (`nestjs-zod`), pino logging.
- Frontend: Next.js 16 App Router, React 19, Tailwind CSS v4, Motion (`motion/react`).
- AI: Vercel AI SDK (`ai` package) with provider adapters. Embeddings via `@huggingface/transformers` in-process (no key). See ADR-0007.
- Python appears nowhere, including scripts. Shell scripts are Bash (Git Bash on Windows) and must also run on Linux CI.

## Consequences
- Breadth across three keywords (NestJS, Next.js, AI SDK) rather than language breadth.
- One toolchain to document and one CI template.
- In-process embeddings cost CPU on Render free; corpus must stay small (< 5k chunks). Acceptable for a demo.
