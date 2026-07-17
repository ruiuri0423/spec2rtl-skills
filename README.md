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
| [`block-spec`](skills/block-spec/) | Detail | Decomposes each hardware block into sub-modules, giving every sub-module a functional contract and an explicit top-down PPA budget — timing, throughput, area, state — plus a verification obligation named before any code exists, so RTL can be written and verified bottom-up. |

Planned, not yet in the library: **rtl-gen** (block specs → RTL), **review**, **verify**.

## How the stages join — the ledger

The load-bearing interface between the skills is not a data format but a discipline. `spec-recovery`
deliberately does **not** decide the hardware; instead it records each such decision as a tagged
open question in a ledger that travels with the spec. Every entry names the stage that will close
it — `behavior`, `reference-defect`, `hardware-binding`, or `must-define-for-hardware` — with a
status. `hardware-spec`'s inbox is exactly the entries tagged `hardware-binding` and
`must-define-for-hardware`; it closes them using requirements the code never contained, and then
emits its own ledger for the block-spec stage. So no stage re-reads another's whole output — each inherits
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
         │   hardware architecture spec  ──(micro-arch ledger entries)──┐
   ┌─────▼─────┐   each block decomposed into sub-modules: contracts,    │
   │ block-    │   top-down PPA budgets that must sum, and     ◄─────────┘
   │ spec      │   verification obligations named before any code
   └─────┬─────┘
         │   block implementation spec  (coding-level choices deferred)
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

This is a **boundary-definition and architecture-mapping** skill, not a code-analysis one: the
information that decides the hardware is not in the code — it lives on the hardware frame's
**boundary**, as concrete properties of concrete ports, and its primary source is the **designer**.
The stage runs in two phases. **Phase A** starts from the functional spec's hardware-ization scope
and defines the frame's complete interface contract — every port enumerated, each mapped to the
functional-spec part it serves, each property (width per beat, rate and streaming discipline,
backpressure, clock owner, update discipline) elicited as a fact of that port, cross-checked once
against reference-implied figures at a reconcile step; no internal structure is touched. **Phase B**
architects within the settled boundary: the default is to map the functional spec directly,
departing only where a requirement's relation forces it — parallelize for throughput, fold for
area, add storage for feedback (bounded below by the loop's **iteration bound**) — quantifying what
the chaining alone demands, and reopening a port through the ledger (with the numbers) if
quantification proves it infeasible. It stops at architecture and its rationale, deferring the
micro-architecture (and the RTL) to the next stage.

## `block-spec` — budget it down to sub-modules

This is a **decomposition and budget-allocation** skill: it turns each hardware block into the
specs RTL will actually be written against. Requirements flow **top-down** as allocated budgets —
the combinational time a sub-module may spend inside the clock, the initiation interval it must
sustain, its share of the block's area envelope, the state it owns — with the arithmetic shown and
every allocation required to **sum** to its envelope. Each sub-module gets a **contract**: its
function restated whole from the functional spec, its interface pinned, its `Budget:` line
(re-derived by a verify pass, like hardware-spec's `Parameters:`), and a **verification
obligation** — the golden evidence that will prove the implementation, named before any code
exists. The point is to make implementation **bottom-up provable**: each sub-module verified
against its own contract in isolation, so integration assembles proven pieces instead of
discovering failures. Tools (generators, DSLs, synthesis probes) are instruments for evidence and
technique, never part of the delivered flow — the deliverables are documents and source a team
maintains. It stops at contracts and budgets, deferring coding-level choices (FSM encoding,
register placement, reset detail) to the RTL stage.

## Shared design principles

- **Prose specs a person can read start to finish** — meaning before representation.
- **The question form follows where the answer lives** — *evidence* questions (what the reference
  means) converge one at a time, each with a recommended reading; *facts* the code cannot contain
  (the boundary's port properties) are gathered by complete enumeration, never one highlighted
  question, with reference-implied figures demoted to a post-answer cross-check; *trades* are
  adjudicated one at a time with both accounts at equal rigor.
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
- **`block-spec` — complete, not yet exercised.** `SKILL.md` and both references —
  `references/method.md` (the budget algebra, the decomposition rule, contracts and verification
  obligations, the tools-as-instruments boundary, and the verify pass) and
  `references/spec-template.md` (the output skeleton) — are written and mutually consistent. It has
  not yet been run end to end on a design; a manual stage-3 try run on the dual-view EOTF block
  (two RTL implementations, exhaustive golden, formal equivalence, PPA measurement) informed the
  verification-obligation and tools-as-instruments sections.

## Revision log

Newest first — what changed in this document and why. The sections above always read as the
current truth; this log is where the history lives.

- `2026-07-17` — **Boundary-first restructuring** (designer-directed): elicitation had been framing
  requirement questions with figures from the functional spec (asking throughput via resolution),
  importing the reference's blind spots — every real requirement failure of the dual-view cycle was
  a boundary fact the reference could not contain. Now: spec-recovery closes with two
  **hardware-ization scope** questions (what goes to hardware; how it sits in the surrounding
  system); hardware-spec splits into **Phase A** (the frame's interface contract — ports enumerated,
  per-port property sheet, facts gathered by enumeration rather than one-highlighted-question,
  reference-implied figures demoted to a post-answer reconcile step) and **Phase B** (blocks and
  chaining quantified within the settled boundary, with a ledger reopen path back to the contract).
  Question forms codified: facts → enumerate; evidence → converge with a recommendation; trades →
  adjudicate at equal rigor.
- `2026-07-16` — hardware-spec hardened with the lessons of a live designer-review cycle on the
  dual-view design (two mis-elicited requirements survived a whole stage; a stance was overturned
  by the designer's deeper accounting; a schedule off-by-one escaped inspection): read-backs must
  state numbers, units, and owners, and never anchor on superseded documents; the binding
  throughput rate is the unit-of-work **formation rate during a burst**, never the average, and
  the formation schedule is derived when the delivery unit differs from the unit of work; storage
  is budgeted at **physical organization** (ports, banks, word width, value domain), never at
  live-data peaks; schedule claims are verified by **cycle-accurate simulation**, not inspection;
  competing stances are rejected only at the winner's rigor; the backpressure question is asked,
  never defaulted (the valid/ready default is removed); the designer may overturn stances
  (challenge → re-derivation, pinned facts recorded), and the reference model is authoritative on
  the computation domain. Checklist, method, and template updated together.
- `2026-07-15` — Added `block-spec`, the third stage: hardware blocks decomposed into sub-modules,
  each with a functional contract, a top-down PPA budget that must sum (the `Budget:` line, verify
  re-derived), and a verification obligation named before any code — so RTL proceeds bottom-up
  against specs that already close. Decided after a manual EOTF try run showed RTL-first jumps
  straight past the questions only a spec can answer (which structure a budget forces — e.g. an
  86-level search cannot close a 2 ns clock — and what each piece may spend). Generator/DSL tools
  are instruments for evidence and technique, never the delivered flow: deliverables stay
  human-maintainable documents and source. hardware-spec's downstream ownership split accordingly:
  structural micro-architecture (`micro-arch`) to block-spec, coding-level to the RTL stage.
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
