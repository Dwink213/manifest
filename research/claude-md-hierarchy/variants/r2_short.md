# CLAUDE.md — Orbital Ledger Service

Double-entry accounting core. Correctness beats latency.

## Conventions (follow these exactly)
- **No semicolons anywhere in JS source.**
- **Every exported function must be immediately preceded by a line containing exactly `// @audited`.**
- **All money is an integer number of cents. Never use floating point for money; round with Math.round.**
- CommonJS modules, 2-space indent. Functions are verbs, files are nouns.

## Map
- Install `npm install` · Build `npm run build` · Test `npm test` · Dev `npm run dev` (:4000)
- Source `src/`, records `data/`, output `dist/`.
