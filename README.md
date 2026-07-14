# spec2rtl-skills

A library of **Claude Code skills** for carrying a hardware design from a reference model toward
RTL, one human-in-the-loop stage at a time. Each skill is self-contained and callable on its own;
together they form a pipeline, joined by a single hand-off artifact — an **open-questions ledger**
— that carries every deferred decision to the stage that will close it. The library is meant to be
pointed at by an agent (or a person) that invokes the internal skills as it works.

*This README was recovered from the repo by the `spec-recovery` method: each skill was read as a
block and understood in its own terms, then the two were reconciled — including an honest account,
below, of what is complete and what is not.*

The conviction shared by every skill here is that **both the artifacts and the skills themselves
must be understandable by a human**: specifications are written as connected prose, not fragmented
tables; uncertainty is surfaced and decided with the designer rather than guessed; and each skill
introduces exactly the structure the task demands and no more.

## The skills

| Skill | Stage | In one line |
|---|---|---|
| [`spec-recovery`](skills/spec-recovery/) | Understand | Reads a reference model (MATLAB / C / C++ / Python) and recovers a plain-language functional specification — what the design does, its real inputs and outputs, its behavior — before any hardware work, deferring every representational choice. |
| [`hardware-spec`](skills/hardware-spec/) | Architect | Turns that functional spec, plus the **designer's** hardware requirements, into a hardware architecture specification — resolving the deferred bindings and introducing only the parallelism, folding, storage, and interfaces a requirement demands. |

Planned, not yet in the library: **rtl-gen** (architecture → RTL), **review**, **verify**.

## How the stages join — the ledger

The load-bearing interface between the skills is not a data format but a discipline. `spec-recovery`
deliberately does **not** decide the hardware; instead it records each such decision as a tagged
open question in a ledger that travels with the spec. Every entry names the stage that will close
it — `behavior`, `reference-defect`, `hardware-binding`, or `must-define-for-hardware` — with a
status. `hardware-spec`'s inbox is exactly the entries tagged `hardware-binding` and
`must-define-for-hardware`; it closes them using requirements the code never contained, and then
emits its own ledger for the RTL stage. So no stage re-reads another's whole output — each inherits
precisely the decisions it owns. The bus runs both ways: a stage may close only the entries tagged
for it, and a question uncovered outside its authority is routed back to its owner rather than
decided quietly.

How all of this *runs* — schedules of tasks with dependencies, stepwise or concurrent execution,
barriers, honest degradation on a limited harness, and the two-way ledger routing — is defined once,
at library level, in the [`pipeline`](skills/pipeline/) skill. The skills declare their schedules; any agent, in
any harness, executes them at whatever capability it has.

```
   reference model
         │
   ┌─────▼─────┐   what the design does, in prose, + a ledger of
   │ spec-     │   deferred decisions, each tagged with the stage
   │ recovery  │   that will close it
   └─────┬─────┘
         │   functional spec  ──(hardware-binding / must-define ledger entries)──┐
   ┌─────▼─────┐   the hardware's shape and the reason for it:                    │
   │ hardware- │   requirement-driven, top-down; departs from the      ◄──────────┘
   │ spec      │   software mapping only where a requirement forces it
   └─────┬─────┘
         │   hardware architecture spec  (micro-architecture deferred)
      (rtl-gen …)
```

## `spec-recovery` — recover what the design does

It reads one reference file and states, in prose, what the block does — naming its purpose, its
real inputs and outputs (including the **hidden** ones the signature omits: files it loads, global
tables, constants baked in, values it computes but drops), and the rules that turn inputs into
outputs. Two passes: the first understands faithfully, the second interrogates for gaps. Anything
uncertain goes through a **three-route triage** — is the answer in a related file? in a known
definition or standard? — and only what neither settles becomes a genuinely open question, put to
the designer one at a time with a recommended default. It captures *meaning*, not representation:
bit widths, ordering, and interface form are named as deferred, never pinned. A whole multi-file
design is handled by parallel sub-agents — draft each block, then aggregate and converge at a
barrier into **one document** with one merged ledger, before anything is resolved with the designer
— so every decision has exactly one home and no write-back pass exists. It does no hardware
partitioning, no micro-architecture, and no RTL.

## `hardware-spec` — give it a hardware shape

This is a **requirements-elicitation and architecture-mapping** skill, not a code-analysis one: the
information that decides the hardware — throughput, clock, area, interface, integration context —
is not in the code, so its primary input is the **designer**. It asks a fixed requirement checklist
one convergent question at a time, uses the answers to close the functional spec's deferred
bindings, and works out each block's architecture. The default is to map the functional spec
directly; it departs only where a requirement forces it, and names the requirement behind every
departure — parallelize for throughput, fold for area, add storage for feedback (bounded below by
the loop's **iteration bound**), define a protocol at each interface — then partitions into
hardware blocks. It stops at architecture and its rationale, deferring the micro-architecture (and
the RTL) to the next stage.

## Shared design principles

- **Prose specs a person can read start to finish** — meaning before representation.
- **A convergent conversation, never a wall of choices** — open points are put to the designer one
  at a time, each carrying a recommended answer.
- **An open-questions ledger that travels with the spec** — every entry tagged and given a status,
  so each downstream stage inherits exactly the decisions it must close.
- **Whole designs in parallel, one deliverable per stage** — drafted block-by-block in parallel,
  then aggregated into a single document (one body, one ledger, one Revision log) at the converge
  barrier, before any question is resolved — so a resolution is one edit, never a synchronized
  sweep across files.

## Using the skills

Each folder under `skills/` is a standard Claude Code skill (a `SKILL.md` plus, where present, its
`references/`). To make a skill available, place its folder under a skills directory — for one
project at `<project>/.claude/skills/<skill>/`, or for every project at `~/.claude/skills/<skill>/`
(copy it, or symlink it to this repo to track updates). Then invoke it by name — for example
`/spec-recovery <reference-file>` — or let an agent call it. An agent driving the whole flow runs
`spec-recovery` first, then `hardware-spec` once a functional spec exists.

## Status

- **`spec-recovery` — complete and validated.** `SKILL.md`, `references/method.md`, and
  `references/spec-template.md` are written, mutually consistent, and have been exercised on three
  unrelated codebases (a MATLAB JPEG pipeline, a Python game, and a Python dual-view image
  pipeline — the last on two different harnesses). The single-document aggregation flow has been
  A/B-validated against the per-block-file baseline on the dual-view design: with the
  carry-substance rule in force, −14% tokens and −37% wall clock versus baseline, all quality
  findings reproduced, and the assembled documents pass the barrier form-check.
- **`hardware-spec` — complete and exercised once.** `SKILL.md` and all three references —
  `references/requirements-checklist.md` (the requirement set and each requirement's convergence
  relation), `references/method.md` (obtaining the parameters, composing the transformations, the
  obligations, and the verify pass), and `references/spec-template.md` (the output skeleton) — are
  written and mutually consistent. Run end to end three times on the dual-view design (concurrent
  harness; once per-block-file, twice single-document); the verify pass confirmed every block each
  run, caught one ledger mis-routing, and — as the barrier form-check — confirmed the assembled
  document carries every derivation.

## Revision log

Newest first — what changed in this document and why. The sections above always read as the
current truth; this log is where the history lives.

- `2026-07-14` — Aggregation hardened: **merging is reconciliation, never summarization.** The
  first A/B re-run of the single-document flow (Dual View, −25% tokens / −44% wall clock, quality
  findings all reproduced) exposed one regression: the hardware document compressed each block's
  derivation to its conclusions and cited the non-surviving drafts as the home of the detail —
  leaving the arithmetic behind every relation uncheckable. The `pipeline` skill now requires the
  assembled document to be checkable with the drafts gone; hardware-spec's template and method
  require each block's section to carry the `Parameters:` line *and* its written-out arithmetic,
  with a form re-check of the assembled document at the barrier; spec-recovery's converge step
  states that merging removes duplication, never substance.
- `2026-07-14` — Each stage's deliverable is now **one document**, aggregated at the converge
  barrier; the write-back task is removed. Driven by the first end-to-end runs (Dual View, two
  harnesses): with per-block specs as deliverables, ledger entries were duplicated across files
  (one question appearing in up to six documents), so every resolution became a synchronized
  multi-file sweep, and a write-back agent existed purely to keep the copies consistent (~9% of
  stage tokens, ~36% of wall clock in the measured run). Fan-out is unchanged — per-block work
  still parallelizes — but its outputs are now drafts merged at the barrier, *before* the designer
  conversation; the `pipeline` skill now defines this aggregation point for every stage.
- `2026-07-13` — Removed the redundant root `PIPELINE.md` pointer; the coordination contract's
  single home is the [`pipeline`](skills/pipeline/) skill.
- `2026-07-13` — Added `PIPELINE.md`, the library-level coordination contract: task-graph
  scheduling, stepwise/concurrent execution modes with one loop, honest degradation, and the two-way
  ledger bus with stage-authority routing. Both skills now declare schedules and delegate execution
  to it; hardware-spec gained the mandatory per-block `Parameters:` line, the boundary
  (never-resolve-outside-your-tags) obligation, and a verify check that each cited relation was
  actually computed.
- `2026-07-13` — Both skills reconstructed into coherent documents (SKILL.md slimmed to a
  contract/overview, procedures consolidated in `method.md`); `hardware-spec` completed with its
  three references. Status updated: `hardware-spec` is now complete (not yet exercised).
- `2026-07-13` — Removed the orphaned `profile-schema.yaml` from `spec-recovery` (designed but never
  wired into the method) and updated the status accordingly; its references are now `method.md` and
  `spec-template.md`. README recovered over the whole repo, and this Revision log added.
