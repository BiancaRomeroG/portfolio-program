# ADR-0009: Documentation and PR standard

- Status: accepted
- Date: 2026-10-28

## Context
The owner requires everything documented: plan, architecture, decisions, PRs, operations. Documentation is also the artifact recruiters and interviewers actually read.

## Decision
1. Every repo contains the file set listed in `PLAN.md` section 4. Missing file = gate fails.
2. ADRs use MADR (title, status, date, context, decision, consequences, alternatives). Numbered per repo starting at 0001. `docs/decisions-log.md` indexes them.
3. `docs/architecture.md` has C4 level 1 (context) and level 2 (containers) as Mermaid diagrams, plus sequence diagrams for the 2–3 key flows, plus a "where things live" table mapping concepts to folders.
4. OpenAPI generated from NestJS Swagger decorators on every build and committed at `docs/api/openapi.json`; CI fails if it is stale (`git diff --exit-code`).
5. `docs/runbook.md`: quickstart, seed/reset, env table, common errors, deploy steps, rollback.
6. `docs/demo.md`: 2-minute click path and the script used for the README GIF.
7. `docs/case-study.md`: problem, constraints, architecture, decisions and trade-offs, results with numbers, what I would do differently. This text feeds the portfolio page.
8. PRs: template mandatory (context, change, test evidence, screenshots, docs checklist, ADR link, risk). Squash merge; PR title is the changelog line.
9. Issues: every task from the spec is an issue with the spec task ID in the title; PRs close issues.
10. Code comments explain *why*, never *what*. Public functions in `packages/shared` have TSDoc.
11. README structure is fixed by `templates/README.template.md`: hero GIF, one-paragraph pitch, "why I built this", architecture image, quickstart (3 commands), features, tech, docs links, status/roadmap, Spanish section, license.

## Consequences
More writing per PR; much better repos. Workers budget ~20% of effort to docs.
