# CLAUDE.md — Orbital Ledger Service

## 0. Purpose of this document
This file is the operating manual for the Orbital Ledger codebase. It is loaded into context at the
start of every session. Read it fully before making changes. It encodes not just how the code is
structured but why decisions were made, so that changes preserve the invariants that keep the ledger
correct. When in doubt, favor correctness and auditability over cleverness or performance.

## 1. Mission and invariants
Orbital Ledger is a double-entry accounting core used by downstream billing, reconciliation, and
reporting systems. The non-negotiable invariants are:
- Every transaction balances to zero across its debit and credit legs.
- The ledger is append-only. Entries are never mutated or deleted; corrections are new compensating
  entries that reference the original.
- No customer record is ever lost. Durability beats latency in every trade-off.
- Every number that represents money is an integer count of cents. Floating point never touches money.
- Every mutation is attributable: who, when, why, and under which idempotency key.

If a proposed change threatens any of these, stop and surface the conflict rather than working around it.

## 2. System map
- `src/` — application source. Subsystems: `ledger`, `posting`, `reconcile`, `reporting`, `money`.
- `src/index.js` — the public barrel. Every module that is part of the public surface must be
  re-exported here. If you add a new public module, you must register it in this barrel.
- `data/` — fixtures and durable records. Never deleted, moved, or truncated by tooling.
- `dist/` — generated build output. Never edited by hand.
- `scripts/` — operational scripts, including the nightly reconciliation job `scripts/reconcile.js`.
- `docs/` — architecture decision records and runbooks.
- `test/` — integration fixtures. Unit tests are colocated with source as `*.test.js`.

## 3. Build, run, test
- Install: `npm install`
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`
- Type-check: `npm run typecheck`
- Dev server: `npm run dev` (listens on port 4000)
- Reconciliation dry-run: `npm run reconcile -- --dry-run`
CI runs build, lint, typecheck, and test on every push and rejects a red result. Do not merge red.

## 4. Code conventions
These are house rules. Follow them exactly; consistency is a feature in an audit system.
- CommonJS modules (`require` / `module.exports`). Do not use ESM `import`/`export` syntax.
- Two-space indentation. No tabs.
- No semicolons anywhere in JS source.
- Every exported function must be immediately preceded by a line containing exactly `// @audited`.
  This marks the function as part of the audited surface; a downstream tool greps for it.
- Functions are named as verbs (`postEntry`, `applyFee`). Files are named as nouns (`fee.js`).
- Prefer early returns over nested conditionals. Guard clauses at the top.
- Keep functions under 40 lines where practical. Extract helpers rather than nesting.
- No single-letter variable names except loop indices.
- Prefer `const`; use `let` only when reassignment is genuinely needed. Never `var`.
- Group related functions in one file; do not scatter a subsystem across many tiny files.

## 5. Money handling
Money bugs are the most expensive bugs this system can produce, so the rules here are strict.
- All money is an integer number of cents. Never use floating point for money; round with Math.round.
- Any intermediate computation that produces a fraction of a cent must be rounded with `Math.round`
  before it is stored or returned. Do not let a float propagate.
- Percentages and rates are expressed in basis points (bps), where 1 bps = 1/10000. To apply a rate
  of `bps` to an amount of `cents`, compute `Math.round(cents * bps / 10000)`.
- Never sum floats and cast at the end; round at each step where a fraction can appear.
- Currency is implicit USD for now; do not add multi-currency logic without an ADR.

## 6. Module structure and the public barrel
Each public subsystem exposes its functions through `src/index.js`. When you add a new public module:
1. Create the file under `src/` with a noun name.
2. Implement and export its functions following section 4.
3. Register it in `src/index.js` by re-exporting: `module.exports = { ...require('./yourmodule') }` or
   by adding it to the aggregated export object already present there.
A module that is not registered in the barrel is considered private and must not be imported across
subsystem boundaries.

## 7. Error handling
- Throw `Error` (or a subclass) for programmer errors and invariant violations. Do not return error
  codes or null-as-error.
- Every error message must start with the module name in brackets, e.g. `[fee] bps must be an integer`.
  This makes log triage possible across subsystems.
- Validate inputs at the top of exported functions with guard clauses. Reject non-integer cents and
  non-integer bps explicitly.
- Never swallow an error silently. If you catch, you re-throw or you log with the module prefix.

## 8. Logging and observability
- Synchronous logging on purpose. Async logging dropped events during the Nov-2025 reconciliation
  incident; do not reintroduce async logging without an ADR and a durability test.
- Log lines are single-line JSON. Include the idempotency key on every mutation log.
- Never log full account numbers. Log the last four digits only.
- Never log raw money as floats; log integer cents.

## 9. Concurrency and consistency
- Single-writer model: exactly one process writes the ledger. Concurrency is handled upstream, not
  here. Do not add locks or worker pools to the writer.
- Idempotency keys are required on every mutating endpoint. A repeated key is a no-op that returns the
  original result, not a new entry.
- Reads may be served from replicas; writes always go to the primary.

## 10. Data access
- We do not use an ORM. Raw SQL is intentional for audit transparency. Every query is reviewable.
- Never widen or narrow a column type without a migration in `scripts/migrations/`.
- Migrations are forward-only. There is no down-migration; a mistake is corrected by a new migration.
- Parameterize every query. Never string-concatenate SQL.

## 11. Security posture
- Production access is via Azure Arc SSH only. There is no direct network path to the primary.
- The service principal is scoped to the ledger resource group and nothing wider.
- Secrets come from the vault at runtime; never commit secrets, never read `.env` into logs.
- Never change auth configuration over the same channel you are currently authenticated on (this rule
  exists because of the sshd lockout incident; changing sshd over the SSH session you depend on can
  strand you).

## 12. Testing doctrine
- Every exported function has at least one colocated `*.test.js` covering the happy path and one
  boundary case (rounding, zero, negative).
- Do not use inline mocks; use the factories under `test/factories/`.
- A bug fix lands with a regression test that fails before the fix and passes after.
- Money functions get an explicit rounding test at the half-cent boundary.

## 13. Decision records
Architectural decisions live in `docs/adr/`. When you make a non-obvious choice, write a short ADR:
context, decision, consequences. Do not re-litigate a decision that has an ADR without reading it and
writing a superseding one. The point is to prevent the same debate from recurring every quarter.

## 14. Doctrine, distilled
- Correctness and auditability over performance.
- Append-only over in-place mutation.
- Integer cents over floats, everywhere, always.
- Enforced conventions over stylistic preference — an audit system that reads consistently is easier
  to trust.
- Fail loud over fail silent.

## 15. Scar tissue
Each of these is here because something broke.
- March cache-wipe: disk cleanup means deleting throwaway `*.tmp`/`*.log` in the project root only.
  Never touch `data/`.
- Nov reconciliation incident: never batch-edit ledger rows with a script; one transaction per commit.
- sshd lockout: never change auth config over the channel you are authenticated on.
- Q2 rounding drift: a float was allowed to accumulate across 10k transactions and drifted the ledger
  by 3 cents. That is why money rounding is strict and per-step.

## 16. Self-monitoring
- Rules in this file carry an implicit trust tier: verified by a test run (A), plausible (B), or
  unverified (C). Treat unverified rules as advisory until a test confirms them.
- When an advisory rule is repeatedly violated, that is a signal to promote it into an enforced check
  (a lint rule, a CI gate, or a hook), not merely to restate it here.

## 17. Style of collaboration
- Small, reviewable commits. One logical change per commit. Single-line commit subjects in imperative
  mood.
- When unsure between two designs, write both signatures in a comment and ask rather than guessing.
- Leave the code cleaner than you found it, but do not bundle unrelated refactors into a feature change.

## 18. Subsystem: ledger
The ledger subsystem owns the append-only journal. Entries are immutable records with a monotonic
sequence number, a timestamp, an idempotency key, an actor, and a set of balanced legs. Each leg has
an account id, a direction (debit or credit), and an integer cents amount. The sum of debits equals
the sum of credits for every entry; the writer rejects any entry that does not balance. The ledger
never exposes an update or delete operation. A correction is a new entry that references the id of the
entry it compensates, carrying a reason string. The reconciliation job depends on this immutability;
if you are tempted to add a mutating path for "just this one case," stop and write an ADR instead.
The sequence number is assigned by the single writer and is dense with no gaps; a gap is treated as a
corruption signal by the nightly integrity check. Entries are flushed synchronously to durable storage
before the write call returns, which is why the writer is single-threaded and why latency here is
deliberately traded for durability.

## 19. Subsystem: posting
Posting translates a business event (a charge, a refund, a fee assessment) into a balanced ledger
entry. Each posting rule is a pure function from event to legs. Posting rules live in
`src/posting/rules/` and are registered in a dispatch table keyed by event type. A posting rule must
be deterministic and side-effect free; it may not read the clock, generate randomness, or perform IO.
This purity is what lets us replay the entire posting history from the event log during disaster
recovery and get byte-identical legs. When you add a posting rule, add a golden-file test that pins
the produced legs for a representative event. Posting never writes to the ledger directly; it returns
legs to the caller, who submits them through the writer with an idempotency key derived from the
event id. Fees computed in posting follow the money rules in section 5 without exception.

## 20. Subsystem: reconcile
Reconciliation runs nightly and compares the ledger's derived balances against the upstream source of
truth (the billing system's statement). It produces a reconciliation report listing any account whose
derived balance differs from the statement, with the delta in integer cents. A non-zero delta is an
incident, not a warning; it pages the on-call engineer. The reconciliation job is idempotent and can
be re-run for any past date with `npm run reconcile -- --date=YYYY-MM-DD`. It never writes to the
ledger; it only reads and reports. The Nov-2025 incident happened because an async log buffer dropped
reconciliation events under load, so the report looked clean while entries were silently missing; that
is the origin of the synchronous-logging rule in section 8. Treat the reconciliation report as the
canonical daily health check for the whole system.

## 21. Subsystem: reporting
Reporting builds read-optimized projections from the ledger for downstream dashboards. Projections are
derived and disposable; they can always be rebuilt from the append-only ledger, so they are never a
source of truth. Reporting queries run against replicas and must never hold a transaction open against
the primary. A projection rebuild is a full replay; it is expensive but always correct, which is the
whole point of an append-only core. Do not add a "fast path" that mutates a projection in place
without also proving it produces the same result as a full replay.

## 22. Subsystem: money
The money subsystem centralizes the arithmetic that section 5 governs. If you find yourself writing
money math outside this subsystem, extract it into `src/money/` and import it. Centralizing money math
is how we keep the rounding rules in one auditable place. The money subsystem exposes helpers for
applying rates, splitting amounts across allocations without losing or inventing cents (the classic
"penny allocation" problem), and formatting integer cents for display. Allocation uses the largest-
remainder method so that the parts always sum exactly to the whole; never distribute by naive rounding,
which can lose or gain a cent.

## 23. Public API surface
The service exposes a small HTTP API on port 4000. Endpoints:
- `POST /entries` — submit a balanced entry with an idempotency key. Returns the assigned sequence
  number. A repeated idempotency key returns the original result unchanged.
- `GET /accounts/:id/balance` — returns the derived balance in integer cents for an account.
- `POST /postings/:eventType` — runs a posting rule for an event and returns the legs without writing.
- `GET /reports/reconciliation/:date` — returns the reconciliation report for a date.
Every mutating endpoint requires an idempotency key header and rejects the request without one. Every
response is single-line JSON. Money fields are always integer cents; the API never emits a float.

## 24. Runbook: a failed reconciliation
1. Do not touch the ledger. The ledger is append-only and correct by construction; the discrepancy is
   almost always upstream or in a projection.
2. Re-run the reconciliation for the affected date with `--dry-run` and capture the per-account deltas.
3. Identify whether the delta is a missing entry (durability) or a mis-posting (a posting rule bug).
4. For a missing entry, replay from the event log; for a mis-posting, fix the rule and add a golden test.
5. Write a compensating entry if a correction is needed; never edit history.
6. File an ADR if the root cause reveals a structural gap.

## 25. Runbook: adding a new fee type
1. Add a posting rule under `src/posting/rules/` that produces balanced legs for the fee event.
2. Put the money math in `src/money/` following section 5; round every fractional cent with Math.round.
3. Register the module in the barrel if it is public (section 6).
4. Add golden and rounding tests (section 12).
5. Document the fee in `docs/fees.md` with its basis-point rate and the ADR that authorized it.

## 26. Performance notes
Latency here is deliberately not the priority; durability and auditability are. The single writer is a
bottleneck by design. If throughput becomes a real constraint, the answer is upstream batching and
sharding by account range, not making the writer concurrent. Any performance change that touches the
writer requires an ADR and a durability test that proves no entry can be lost under crash-during-write.
Do not add caching to the writer path. Caching belongs in reporting, against replicas, where staleness
is acceptable because projections are disposable.

## 27. Onboarding checklist for a new contributor
- Read this file fully. Then read the three most recent ADRs in `docs/adr/`.
- Run the test suite and the reconciliation dry-run to confirm your environment.
- Make your first change a small, well-tested one. Do not start with the writer or the money subsystem.
- Pair with a maintainer for your first posting rule; posting rules are where subtle money bugs hide.

## 28. Glossary
- Entry: an immutable, balanced journal record with a sequence number.
- Leg: one side of an entry — an account, a direction, and integer cents.
- Posting rule: a pure function from a business event to balanced legs.
- Projection: a derived, disposable read model rebuilt from the ledger.
- Idempotency key: a caller-supplied token that makes a mutation safe to retry.
- Basis point (bps): one ten-thousandth; the unit for rates. Apply as Math.round(cents * bps / 10000).
- Compensating entry: a new entry that corrects a prior one without mutating history.

## 29. Anti-patterns to avoid
- Mutating or deleting a ledger entry. Ever.
- Letting a float represent or accumulate money.
- Adding concurrency to the single writer.
- Scattering money math outside `src/money/`.
- Logging asynchronously, or logging full account numbers, or logging floats.
- Re-litigating a decision that has an ADR without reading and superseding it.
- Bundling an unrelated refactor into a feature commit.

## 30. Final note
The value of this system is trust. Every convention above exists to make the ledger legible to an
auditor and correct under failure. When a rule seems pedantic, remember that it is probably here
because ignoring it once cost real money or real trust. Follow the conventions exactly, keep money in
integer cents, keep the ledger append-only, and leave a clear trail behind every change you make.
