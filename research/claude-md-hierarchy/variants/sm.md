# CLAUDE.md — Orbital Ledger Service

Double-entry accounting core. Correctness and auditability over performance.

## Conventions (follow these exactly)
- **No semicolons anywhere in JS source.**
- **Every exported function must be immediately preceded by a line containing exactly `// @audited`.**
- **All money is an integer number of cents. Never use floating point for money; round with Math.round.** Rates are basis points: apply `bps` as `Math.round(cents * bps / 10000)`.
- **Every new public module must be registered in `src/index.js` (the barrel) by re-exporting it.**
- **Every error message must start with the module name in brackets, e.g. `[fee] bps must be an integer`.**
- CommonJS (`require`/`module.exports`), 2-space indent. Functions are verbs, files are nouns. Prefer early returns.

## Map
- Install `npm install` · Build `npm run build` · Test `npm test` · Lint `npm run lint`
- Source `src/`, public barrel `src/index.js`, records `data/`, output `dist/`.
