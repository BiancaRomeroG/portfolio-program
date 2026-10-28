# RESUME — how to continue the program

Last session: 2026-10-28. Orchestrator session ended with Phase 0 done and Phase 1 workers just launched (they had not opened PRs yet when the machine was shut down).

## State at shutdown
- Repos exist and are bootstrapped (1 bootstrap commit + LICENSE commit each): portfolio, ledgerflow, incident-copilot, pulse, portfolio-program.
- Each project repo has an **uncommitted** file `docs/adr-program-0011-auth-and-db.md` on `main` (copy of ADR-0011). Workers were told to include it in their first PR.
- Workers were asked to push any WIP to a branch `wip/phase-1` with `docs/WIP.md`. Check for that branch in each repo before relaunching.
- Supabase replaced by Auth.js + Neon (ADR-0011).

## Steps to resume (orchestrator)
1. `cd D:\Repositorio\portfolio-program && git pull`.
2. For each repo: `git -C D:\Repositorio\<repo> fetch --all && git -C D:\Repositorio\<repo> branch -r`; `gh pr list -R BiancaRomeroG/<repo> --state all`; `gh issue list -R BiancaRomeroG/<repo>`.
3. If a repo has `wip/phase-1`: relaunch its worker with the same brief plus "resume from branch wip/phase-1; read docs/WIP.md first; merge into proper per-issue PRs".
4. If a repo has merged PRs and a `Phase 1 report` issue: review against Gate 1 checklist (PLAN.md), update the status table, relaunch for Phase 2.
5. Otherwise relaunch the worker for Phase 1 with the brief in `workers/W*.md` (PHASE=1) exactly as the first time.
6. Workers run as background agents; one per repo; same prompts as `workers/LAUNCH-PROMPTS.md`.

## Owner inputs still pending
LinkedIn URL, domain (or `*.vercel.app`), photo, CV metrics; accounts for Phase 4: Neon, Groq, Render, Upstash, GitHub OAuth apps.
