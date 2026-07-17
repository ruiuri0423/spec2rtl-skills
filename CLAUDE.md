# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A library of Claude Code **skills** (markdown only — no build, test, or lint tooling) that carries a
hardware design from a reference software model toward RTL, one human-in-the-loop stage at a time.
The deliverables here are the skill documents themselves; "developing" in this repo means editing
prose contracts and keeping them mutually consistent.

Each skill lives at `skills/<name>/` as a standard Claude Code skill: a `SKILL.md` (YAML frontmatter
with `name` and `description`, then a contract/overview) plus, where present, `references/` holding
the full procedure (`method.md`), the output skeleton (`spec-template.md`), and for hardware-spec
the requirement set (`requirements-checklist.md`). Skills are installed by copying or symlinking a
folder into `<project>/.claude/skills/` or `~/.claude/skills/`.

## The pipeline and its stages

- `spec-recovery` (Understand) — reads a reference model (MATLAB / C / C++ / Python) and recovers a
  plain-language functional spec, deferring every hardware/representational choice (bit widths,
  ordering, interfaces). Closes with two hardware-ization scope questions (what goes to hardware;
  how it sits in the surrounding system). Complete and validated.
- `hardware-spec` (Architect) — two phases, boundary first: Phase A defines the frame's interface
  contract (ports enumerated from the scope section; per-port property sheet elicited as facts,
  reference figures only at a post-answer reconcile step); Phase B architects blocks and chaining
  within the settled boundary, quantified by the convergence relations, with a ledger reopen path
  back to the contract. Complete, validated on the dual-view design.
- `block-spec` (Detail) — decomposes each hardware block into sub-modules, each with a functional
  contract, a top-down PPA budget that must sum (`Budget:` line, analog of hardware-spec's
  `Parameters:`), and a verification obligation named before any code exists — so RTL is built and
  verified bottom-up. Tools (DSLs, generators, synthesis probes) are instruments for evidence, never
  part of the delivered flow: deliverables are human-maintainable documents and source. Complete,
  not yet exercised.
- `pipeline` — not a work stage but the library-level **coordination contract**: schedules as task
  graphs (tasks, dependencies, barriers), one execution loop that runs identically stepwise or
  concurrently, honest degradation on limited harnesses, and the ledger routing rules.
- Planned, not present: `rtl-gen` (block specs → RTL), `review`, `verify`.

## The load-bearing interface: the open-questions ledger

Stages do not exchange data formats; they exchange a **tagged ledger** that travels with every spec.
Each entry names the stage that closes it — `behavior`, `reference-defect`, `hardware-binding`,
`must-define-for-hardware` — with a status (`open` / `resolved` / `deferred`). Two rules, defined in
`skills/pipeline/SKILL.md`, govern the bus:

1. A stage may close **only** entries tagged for it (hardware-spec's inbox is `hardware-binding` +
   `must-define-for-hardware`).
2. A question a stage cannot close is **routed to its owner** — including upstream: hardware work
   that exposes a behavioral ambiguity writes it back into the functional spec's ledger rather than
   deciding it quietly.

Any change to tags, routing, scheduling, or execution modes belongs in the `pipeline` skill (its
single home — a former root `PIPELINE.md` was deliberately removed); the other skills only declare
their schedules in their `method.md` and point at `pipeline` for execution.

## Conventions the documents enforce

- **One deliverable document per stage; aggregation at the barrier.** Fan-out per-block work
  produces *drafts* (returned messages or working files), merged at the stage's converge barrier —
  before any open question is resolved — into a single document with one body, one ledger, one
  Revision log. There is deliberately no write-back task; a decision must never exist in more than
  one file. The `pipeline` skill defines this aggregation point.
- **Prose over tables; drive reasoning, don't dictate output.** Specs and skills are written as
  connected prose a person can read start to finish. Skills state invariants (traceable, reasoned,
  computed-in-the-open) rather than rules about length or wording.
- **Flag, don't fabricate.** Nowhere in the library is guessing acceptable — open points go to the
  ledger or to the designer as a convergent conversation, one question at a time, each with a
  recommended answer.
- **SKILL.md is the contract; method.md is the procedure.** Keep them mutually consistent when
  editing either — the README's Status section explicitly tracks this consistency per skill.
- **Revision logs, newest first.** The README and the spec templates carry a Revision log; sections
  above it always read as current truth, history lives only in the log. Substantive changes to the
  library should update the README's Status and Revision log accordingly.
