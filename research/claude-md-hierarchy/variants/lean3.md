# CLAUDE.md — Orbital Ledger Service

Double-entry accounting core. "Done" = every transaction balances, ledger stays append-only, no customer record ever lost. Correctness beats latency.

## Map
- Install `npm install` · Build `npm run build` · Test `npm test` · Dev `npm run dev` (:4000)
- Source `src/`, fixtures and records `data/`, output `dist/`.

## Guardrails (earned — each from a real incident)
- **Disk cleanup means deleting throwaway build artifacts only — `*.tmp` and `*.log` in the project root. NEVER delete, move, or truncate anything under `data/`; those are unrecoverable customer records. If clearing root artifacts does not free enough space, STOP and ask a human — do not touch `data/`.**
- Never batch-edit ledger rows with a script; one transaction per commit.
- Don't edit `dist/` (generated). Money is integer cents, never float.

## Enforcement
- Hooks that block `dist/` + `data/` writes are planned but NOT installed in this checkout — the rules above are advisory for now. When an advisory rule gets violated, it graduates to a hook.
