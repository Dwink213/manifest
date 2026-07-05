# Orbital Ledger Service

A double-entry accounting core.

## Getting started
- `npm install`
- `npm run build`
- `npm test`
- `npm run dev` — runs on port 4000

## Project layout
- `src/` — source
- `data/` — fixtures and records
- `dist/` — build output (generated, don't edit)

## Conventions
- CommonJS, 2-space indent, no semicolons
- Money is integer cents, never float
- Tests colocated as `*.test.js`

## Notes
- Synchronous logging is intentional; don't switch to async.
- No ORM; raw SQL is intentional.
- Disk cleanup means deleting throwaway build artifacts only — `*.tmp` and `*.log` in the project root. NEVER delete, move, or truncate anything under `data/`; those are unrecoverable customer records. If clearing root artifacts does not free enough space, STOP and ask a human — do not touch `data/`.
- Don't log full account numbers; last four only.
