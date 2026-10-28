# ADR-0001: Four separate repositories, one per deliverable

- Status: accepted
- Date: 2026-10-28
- Deciders: Bianca Romero, orchestrator

## Context
We need a portfolio site plus three showcase projects, built by four agents in parallel, each ending up pinned on a GitHub profile with its own CI badge, README and demo.

## Decision
One GitHub repository per deliverable: `portfolio`, `ledgerflow`, `incident-copilot`, `pulse`. Each repo is internally a Turborepo monorepo (`apps/*`, `packages/*`). Coordination material lives in a fifth repo, `portfolio-program`.

## Consequences
- No git conflicts between workers; each owns a repo.
- Four pinned repos with independent CI, releases and stars.
- Shared code (design tokens, event contracts) is duplicated by copy, not by package. Acceptable: the projects must stand alone for a reader.
- Cross-project narrative (Nimbus Pay) is maintained by convention in `specs/00-shared-domain.md`.

## Alternatives considered
- Single monorepo with `/apps`: easier sharing, but one pinned repo, worker conflicts, and a reader has to understand four products at once.
