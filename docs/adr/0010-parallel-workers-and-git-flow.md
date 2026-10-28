# ADR-0010: Parallel agent workers and git flow

- Status: accepted
- Date: 2026-10-28

## Decision
- Four workers (W0–W3), one per repo, launched by the orchestrator with the briefs in `workers/`. They run concurrently and independently.
- Git flow: trunk-based. Branch per issue, PR with template, CI must be green, squash merge by the worker itself, branch deleted. No direct pushes to `main` except the bootstrap commit. No branch protection rules (the owner may self-merge too), discipline is enforced by `CLAUDE.md` and gate reviews.
- Commits are authored by the owner's git identity (`BiancaRomeroG <biancaromeroguz@gmail.com>`) with the `Co-Authored-By: Claude` trailer, so GitHub shows the collaboration honestly.
- The orchestrator reviews gates, resolves cross-repo contract questions, updates `PLAN.md` status, and relaunches workers for the next phase with a short delta brief.
- Workers report by opening a `Phase N report` issue. No chat-only reporting.

## Consequences
- Full PR and issue history from day one: visible process, not just code.
- Honest attribution of AI assistance, consistent with the CV's "AI-assisted development pipeline" story.
