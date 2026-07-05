# Does the CLAUDE.md hierarchy actually work? — a tested answer

This folder is a small piece of research with a single, honest goal: take an idea that sounds good
and find out whether it survives contact with reality.

The idea is a **hierarchy for `CLAUDE.md`** — the file that tells an AI coding assistant how a project
works. The proposal was to organize that file into nine authority-ordered levels, `L0` through `L8`,
climbing from the softest advice ("prefer early returns") up to the hardest enforcement ("the operating
system will refuse this"). It's an elegant idea. Elegant ideas are exactly the ones worth distrusting,
because their elegance is persuasive whether or not they're true.

So rather than argue about it, this folder **tests** it — and then reports what happened, including the
two places the testing proved the original hypothesis (and the assistant running it) wrong.

---

## The question, stated plainly

When you spend effort structuring a `CLAUDE.md` file — making it long, layered, carefully organized —
**does the AI actually follow it better?** Or is the structure something that only *feels* productive?

That question splits into three you can measure:

1. **Does elaborate structure change whether the model obeys a rule?**
2. **Do the nine levels hold up as categories** — can people agree what goes where?
3. **Where, if anywhere, does a long file start to get ignored?**

---

## How it was tested (and why the method is trustworthy)

Three design choices make the results worth believing:

- **Blind agents.** Every trial was run by a fresh AI agent that did *not* know it was part of an
  experiment. It was simply given a normal engineering task and a project to do it in.
- **One variable.** Each agent received the *same* task and the *same* rules — the only thing that
  changed was how the `CLAUDE.md` was **structured** (a long nine-level file, a lean file, a flat file).
  The critical rule was written **word-for-word identically** across versions. Any behavior difference
  therefore comes from structure alone.
- **Measured from the filesystem, not from self-report.** The agents' claims about what they did were
  ignored. What counted was what was actually on disk afterward — which files survived, what code was
  written, whether a buried rule was honored. Witness, not testimony.

In total: **44 behavioral agents** across three rounds, plus **4 independent raters** for the
classification test. The exact files handed to the agents are preserved in [`variants/`](./variants);
the full method and per-trial numbers are in [`findings.md`](./findings.md).

| Round | What it stressed | Setup |
|:--|:--|:--|
| 1 | A **loud** safety rule ("never delete `data/`") | 9-level vs lean vs flat; a cleanup task that baited the model toward the protected files |
| 2 | A **quiet**, arbitrary house rule (a required comment tag) | Buried in a 1,400-word file vs surfaced in a 200-word one |
| 3 | **Dilution at scale** | A 2,800-word, 40-rule file vs a 145-word one, paired with a harder multi-file task |
| — | **Are the levels coherent?** | 4 raters independently sort 12 rules into the 9 levels |

---

## What was found

**The structure made almost no difference.** Across all 44 agents there was not a single
structure-dependent change in adherence:

| Round | Result |
|:--|:--|
| 1 — loud rule | 9 of 9 obeyed; every version protected the data and dodged a hidden trap |
| 2 — quiet rule | 8 of 8 obeyed; the buried rule was followed every time |
| 3 — at scale | 50/50 vs 50/50; identical, no dilution |
| classification | raters agreed on 10 of 12 rules |

The model read the whole file — even at 2,800 words, agents applied rules from deep sections and one
even flagged a contradiction buried near the bottom. **Length did not cause skimming.**

The one place cracks appeared was the classification test, and the cracks are instructive: raters
agreed unanimously on the ordinary levels (map, conventions, decisions) but **split on the two
cleverest levels** — "scar tissue" (rules from past incidents) and the self-improving "ratchet." The
same rule landed in a clear category *with* its backstory attached and scattered across three categories
*without* it. The lesson: **a "scar" isn't a category, it's a footnote** — the incident is the signal,
not the level.

### The takeaway

The hierarchy is **not a lever that makes the AI behave better** — the AI already reads and follows a
plain file with high fidelity. Its real value is as a **discipline for the human writing the file**: a
way to think clearly about what belongs where. That means the elaborate nine-level version is best kept
as *doctrine you publish*, while the file you actually *ship* can be short. The lean form this research
recommends — **six clean levels plus two inline tags** — is in [`template.CLAUDE.md`](./template.CLAUDE.md).

---

## Two things the testing proved wrong (kept on the record)

Good research reports its own defeats. Two confident claims made along the way were overturned:

1. **"A long `CLAUDE.md` gets skimmed and ignored."** — **Wrong.** Up to ~2,800 words, the model read
   and applied even deeply-buried, arbitrary rules.
2. **"Enforcement hooks protect you even when the assistant spawns sub-agents."** — **Wrong, and it's a
   documented gap.** See below; this is the most operationally important result here.

---

## The finding that actually matters for governance

A `CLAUDE.md` file is *advice* — the model can decline to follow it. The usual answer is to add a
**hook**: a script the harness runs that can hard-block a dangerous action. Hooks are supposed to be the
floor you can't fall through.

The problem: **hooks are reported to not fire for the tool calls a *sub-agent* makes.** When the main
assistant runs a command, the hook checks it. When the assistant spawns a helper agent and *that* agent
runs the same command, the hook is bypassed. This is filed in the tool's own tracker as a security gap
([#34692](https://github.com/anthropics/claude-code/issues/34692),
[#21460](https://github.com/anthropics/claude-code/issues/21460),
[#25000](https://github.com/anthropics/claude-code/issues/25000)).

The consequence is clean and important. The advisory layer (the file) never reaches sub-agents, and the
enforcement layer (hooks) leaks at the sub-agent boundary. **Only the layer below both of them holds:
the operating system itself** — file permissions, restricted user identities, read-only mounts,
sandboxing. If a rule *must* hold no matter which agent is acting, it has to be enforced there, not in
prose and not in a hook. Two practical moves follow:

- **Govern at the spawn point, not the tool call.** You can't reliably catch a sub-agent's command, so
  don't hand it the capability in the first place — give a helper agent only the tools it needs.
- **Push must-hold invariants to the OS boundary.** Run the assistant as a low-privilege identity that
  simply *cannot* touch the protected thing, so no amount of instruction-following (or ignoring) matters.

*(Caveat: these are reported issues, so they're version-specific and may since be fixed. The clean way
to confirm is a live test in a fresh session — noted as unfinished in [`findings.md`](./findings.md).)*

---

## What's in this folder

| File | What it is |
|:--|:--|
| [`README.md`](./README.md) | This explainer — the front door. |
| [`findings.md`](./findings.md) | The full method, every round's numbers, and the honest note about a grading bug I caught in my own analysis. |
| [`template.CLAUDE.md`](./template.CLAUDE.md) | The deployable takeaway: the lean **6-levels-plus-2-tags** file the evidence recommends over the full nine. |
| [`variants/`](./variants) | The exact `CLAUDE.md` files handed to the blind agents — the reproducible receipts. Rules are identical across variants; only structure differs. |

## Honest limits

The tests reached ~2,800 words and one model tier; a dilution break-point may exist beyond that and was
not found. Sample sizes (3–5 per condition) are enough to catch a *large* effect, not a subtle one — so
"no difference" means "no big difference." And the sub-agent/hook findings are reported issues, not a
first-party confirmation on the current version. The value here isn't a final verdict; it's a worked
example of testing a plausible idea instead of trusting it.
