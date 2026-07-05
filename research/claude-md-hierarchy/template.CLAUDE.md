# CLAUDE.md — <PROJECT NAME>

<!--
  DEPLOYABLE TEMPLATE — the lean form of the L0–L8 hierarchy, derived from the research in this folder
  (see README.md and findings.md). The full 9-level ladder is published doctrine; this is the tool.
  Evidence: model adherence is structure-independent up to ~2,800 words, so the taxonomy is for the
  HUMAN author, not the model. Two changes vs. the 9-level design, both earned from the classification
  test:
    • L5 "Scar tissue" was demoted from a LEVEL to a TAG — write "(scar: <incident>)" next to the rule
      it explains. Reason: a scar with its story got unanimously classified; without the story it split
      three ways. It's a footnote, not a category.
    • L6 "Enforcement" and L8 "Ratchet" were merged — promotion is an ACTION of self-monitoring, not a
      level of its own.
  Result: 6 clean levels + 2 inline tags. Keep this file short and high-signal (target < ~200 lines).
  Emphasis ("IMPORTANT", "YOU MUST") on the 1–2 truly critical rules is officially endorsed and works.
-->

## L0 — Invariant (never changes)
<!-- Mission + definition of "done"/"correct" in one paragraph. If you can't state it, you're not ready. -->
<one paragraph>

## L1 — Map (facts; verify with a real run before trusting)
<!-- Build/run/test commands, key paths, ports. This is what the model burns tokens rediscovering. -->
- Install: `...`   Build: `...`   Test: `...`   Dev: `...`
- Source `...`, output `...`, protected data `...`

## L2 — Grammar (conventions)
<!-- Naming, structure, idioms. "Write code that reads like the surrounding code." -->
- ...

## L3 — Doctrine (decisions + the WHY — prevents re-litigation)
<!-- Each entry: the decision, then the reason. The reason is what lets the rule generalize. -->
- <decision> — because <reason>.  (scar: <incident>)   <!-- attach the scar tag where one applies -->

## L4 — Guardrails (advisory — the model CAN ignore these)
<!-- "Don't do X." Use IMPORTANT / YOU MUST on the 1–2 that genuinely matter. Tag scars inline. -->
- **YOU MUST NEVER** <destructive action>. (scar: <incident>)
- Do not <X>.

## L7 — Boundary (what actually enforces — the only layer that holds across sub-agents)
<!--
  MUST-HOLD invariants live here, because advisory text (L0–L4) does not reach sub-agents and hooks are
  reported to not fire for sub-agent tool calls (see README.md, "the finding that actually matters").
  Document the real enforcement:
-->
- Enforcement present: <hooks / settings deny / none yet — be honest about this>
- OS boundary: <read-only mounts, low-priv identity, network policy, IAM scope>
- Sub-agent policy: scope tools at spawn (give a helper only the tools it needs); do not rely on hooks
  to catch a sub-agent's shell command.

## Self-monitoring (the ratchet — a process, live from day one)
<!-- Not a level; a maintenance rule that keeps the file honest. -->
- Trust tags: [A] verified by a run · [B] plausible · [C] unverified. Tag factual claims above.
- Promotion rule: when an L4 guardrail is violated (or a scar recurs), it FAILED as advice — promote it
  to real enforcement in L7 (a hook / settings deny / OS control); don't just restate it here.
