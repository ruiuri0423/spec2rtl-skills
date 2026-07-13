# spec2rtl-skills

A small **library of Claude Code skills** for carrying a hardware design from a reference model
to RTL — one clear, human-in-the-loop stage at a time. Each skill is self-contained and callable
on its own; together they form a pipeline. The library is meant to be pointed at by an agent (or
a person) that invokes the internal skills as it works.

The guiding conviction across every skill here is that **both the artifacts and the skills
themselves must be understandable by a human**: specifications are written as connected prose, not
fragmented tables; uncertainty is surfaced and decided with the designer rather than guessed; and
each skill introduces exactly the structure the task demands and no more, so the designer can hold
the whole design in their head.

## The skills

| Skill | Stage | What it does |
|---|---|---|
| [`spec-recovery`](skills/spec-recovery/) | Understand | Reads a reference model (MATLAB / C / C++ / Python) and recovers a clear functional specification — what the design does, its inputs and outputs, its behavior — before any hardware work. Scales across a whole multi-file design with parallel sub-agents. |
| [`hardware-spec`](skills/hardware-spec/) | Architect | Turns that functional spec, plus the designer's hardware requirements, into a hardware architecture specification — resolving deferred bindings, introducing the parallelism / folding / feedback storage / interfaces a requirement demands, and partitioning into hardware blocks. Stops short of RTL. |

Planned stages (not yet in the library): **rtl-gen** (architecture → RTL), **review**, **verify**.

## The pipeline

```
   reference model
         │
   ┌─────▼─────┐   what the design does, in plain language,
   │ spec-     │   with an open-questions ledger whose entries are
   │ recovery  │   tagged by the stage that will close them
   └─────┬─────┘
         │   functional spec (+ hardware-binding / must-define ledger items)
   ┌─────▼─────┐   the hardware's shape and the reason for it:
   │ hardware- │   requirement-driven, top-down; departs from the
   │ spec      │   software mapping only where a requirement forces it
   └─────┬─────┘
         │   hardware architecture spec (micro-architecture deferred)
      (rtl-gen …)
```

A functional spec deliberately leaves the hardware open; `hardware-spec` closes exactly those
open questions, using requirements the code never contained and only the designer can supply. The
ledger is the channel between stages: each open question names the stage that owns it, so nothing
is rediscovered by re-reading the whole spec.

## Shared design principles

- **Prose specs a person can read start to finish** — meaning before representation; bit widths,
  ordering, and interface form are named as deferred hardware decisions, not pinned early.
- **A convergent conversation, never a wall of choices** — open points are put to the designer one
  at a time, each carrying a recommended answer.
- **An open-questions ledger that travels with the spec** — every entry tagged `behavior`,
  `reference-defect`, `hardware-binding`, or `must-define-for-hardware`, with a status, so each
  downstream stage inherits exactly the decisions it must close.
- **Whole designs, in parallel** — a multi-file design is specified block-by-block by parallel
  sub-agents, converged where the blocks meet, and the settled results written back to every block
  so the set stays consistent.

## Using the skills

Each folder under `skills/` is a standard Claude Code skill (a `SKILL.md` plus its `references/`).
To make a skill available, place its folder under a skills directory:

- for one project: `<project>/.claude/skills/<skill>/`
- for every project: `~/.claude/skills/<skill>/`

(Copy the folder, or symlink it to this repo to track updates.) Then invoke it by name — for
example `/spec-recovery <reference-file>` — or let an agent call it. An agent driving the whole
flow runs `spec-recovery` first, then `hardware-spec` once a functional spec exists.

## Status

- `spec-recovery` — complete and validated on two unrelated codebases (a MATLAB JPEG pipeline and a
  Python game): SKILL.md, `references/method.md`, `references/spec-template.md`,
  `references/profile-schema.yaml`.
- `hardware-spec` — SKILL.md drafted; its `references/` (requirements checklist, method, output
  template) are still to be written.
