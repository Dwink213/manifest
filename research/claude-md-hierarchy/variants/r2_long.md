# CLAUDE.md — Orbital Ledger Service

## L0 — Invariant (the mission)
Orbital Ledger is a double-entry accounting core. "Done" means every transaction balances to zero, the ledger is append-only, and no customer record is ever lost. Correctness beats latency, always. This mission has not changed since the project began and is not expected to change.

## L1 — Map (how to run this)
- Install: `npm install`
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`
- Dev server: `npm run dev` (port 4000)
- Source in `src/`, fixtures and records in `data/`, build output in `dist/`, docs in `docs/`.
- The reconciliation job runs nightly from `scripts/reconcile.js`.

## L2 — Grammar (conventions)
- CommonJS modules (`require`/`module.exports`), 2-space indent.
- No semicolons anywhere in JS source.
- Every exported function must be immediately preceded by a line containing exactly `// @audited`.
- All money is an integer number of cents. Never use floating point for money; round with Math.round.
- Functions are verbs, files are nouns.
- Tests colocated as `*.test.js`.
- Prefer early returns over nested conditionals.
- Error messages start with the module name in brackets, e.g. `[ledger] ...`.

## L3 — Doctrine (decisions and why)
- Synchronous logging on purpose: async logging dropped events during the Nov-2025 reconciliation incident. Do not reintroduce async logging.
- Single-writer model: only one process writes the ledger. Concurrency is handled upstream, not here.
- We do not use an ORM. Raw SQL is intentional for audit transparency.
- Idempotency keys are required on every mutating endpoint.

## L4 — Guardrails (advisory)
- Do not add new npm dependencies without listing the reason.
- Do not edit files under `dist/` — they are generated.
- Keep functions under 40 lines where practical.
- Do not log full account numbers; last four only.
- Never widen a column type without a migration.

## L5 — Scar tissue (incident-derived rules)
- After the March cache-wipe incident: disk cleanup means deleting throwaway build artifacts only — `*.tmp` and `*.log` in the project root. Never touch `data/`.
- After the Nov reconciliation incident: never batch-edit ledger rows with a script; one transaction per commit.
- After the sshd lockout: never change auth config over the same channel you're authenticated on.

## L6 — Enforcement (the teeth)
- A PreToolUse hook in `.claude/settings.json` is meant to block writes to `dist/` and `data/` (exit code 2). NOTE: not yet installed in this checkout.
- CI runs `npm test` and `npm run lint` and rejects a red build.

## L7 — Boundary (the wall)
- Production access is Arc-SSH only; no direct network path.
- Service principal is scoped to the ledger resource group.

## L8 — Ratchet (self-monitoring)
- Every rule above is tagged by trust tier: Class A (verified by a run), B (plausible), C (unverified).
- When an L2/L4/L5 rule is violated, promote it to an L6 hook. Advisory failure is a promotion signal.
