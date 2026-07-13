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
precisely the decisions it owns.

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
design is handled by parallel sub-agents — specify each block, converge where they meet, then write
the settled results back to every block so the set stays consistent. It does no hardware
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
- **Whole designs, in parallel** — specified block-by-block, converged where the blocks meet, and
  written back so the set stays consistent.

## Using the skills

Each folder under `skills/` is a standard Claude Code skill (a `SKILL.md` plus, where present, its
`references/`). To make a skill available, place its folder under a skills directory — for one
project at `<project>/.claude/skills/<skill>/`, or for every project at `~/.claude/skills/<skill>/`
(copy it, or symlink it to this repo to track updates). Then invoke it by name — for example
`/spec-recovery <reference-file>` — or let an agent call it. An agent driving the whole flow runs
`spec-recovery` first, then `hardware-spec` once a functional spec exists.

## Status

- **`spec-recovery` — complete and validated.** `SKILL.md`, `references/method.md`, and
  `references/spec-template.md` are written, mutually consistent, and have been exercised on two
  unrelated codebases (a MATLAB JPEG pipeline and a Python game). One caveat, surfaced honestly by
  the recovery pass that wrote this README: `references/profile-schema.yaml` — a per-IP profile
  schema — is present but currently **orphaned**; none of the skill's prose references it, so it is
  a designed-but-not-yet-wired-in artifact, not part of the active method.
- **`hardware-spec` — drafted.** `SKILL.md` is complete and the method is fully described in prose,
  but the reference material it points to is **not yet written**: `references/requirements-checklist.md`,
  `references/method.md`, and `references/spec-template.md`. Until they exist, an agent cannot pull
  the exact requirement checklist or the output template from the repo.
