# Functional Specification — <block name>

This document is the output of the spec-recovery stage. Its job is to say, in plain
language, what a piece of reference software does — clearly enough that someone who has
never seen the code understands the design, and completely enough that the later stages
can build on it. It stops at *what the block does*. It deliberately says nothing about
hardware: no clock, no interface, no memory organization, no bit widths. Those decisions
are made later, by the skills that read this document. Asking for them here would force a
reader to commit to hardware before they even understand the design well enough to make
those choices — so this stage gathers requirements only, and the requirements are handed
forward to be turned into a hardware specification step by step.

Write every section as prose. Explain not only *what* is true but *why* it is true and
*how* it works, the way you would explain the design to a new colleague. Where the
reference does not settle something, do not guess — write `UNCONFIRMED:` followed by what
is known and what is missing, and carry it down to the open questions at the end.

*Derived from: `<source file>`. Status: draft, pending the open questions below.*

---

## 1. Purpose — what this block is for

Explain, in a few sentences, why this block exists and what problem it solves for the
system around it. Say where it sits: what comes before it, what comes after. State the
boundary of its responsibility plainly — for example, "it produces quantized coefficients,
not pixels" — because a reader's first mistake is almost always about scope, and naming
the boundary early prevents it.

## 2. Inputs — what goes in

Describe everything the block consumes. For each input, say what it represents, what form
it takes, and any ordering or convention that gives it meaning. Include the inputs that are
not obvious from the interface — data the code loads, tables it relies on, constants it
quietly assumes — because a maintainer who does not know these exist cannot use or change
the block. If an input's form is only a convenience of the reference model rather than its
true nature, say so, and describe what it actually represents.

## 3. Outputs — what comes out

Describe everything the block produces, in the same way: for each output, what it means and
how it is organized. Watch for results the code computes and uses internally but never
hands back. If the surrounding system needs such a value, it is an output too, and
recording it here is what lets the block connect cleanly to whatever comes next.

## 4. Resources — what it needs to hold and to compute

Describe, at the level of the algorithm, what the block must *remember* and what work it
must *do*. For what it remembers: which values hold state, how large they are, how they are
indexed, and how long they must live — only within a single run, or carried across runs.
For what it does: the significant operations the algorithm performs and roughly how many of
them, per input, per element, or per block. This is not a hardware budget and names no
gates or registers; it is an honest account of the demands the computation makes, so that a
later stage can size real hardware from something concrete rather than from a guess.

## 5. Behavior — what it does

Explain how the block turns its inputs into its outputs. Walk through the steps in order:
the phases, the loops and what ends them, the decisions and what drives them. Include the
numeric rules that live inside those steps — how signs are handled, how values are scaled
or rounded, and where a result depends on an earlier one. This is the heart of the
specification; write it so that a reader could reconstruct the intended behavior faithfully
without ever seeing the original code.

## Open questions (ledger)

Collect here every point the design still leaves open — what is unresolved, why it matters,
and what would settle it. This is a **ledger that travels with the spec**, not a footnote read
once and lost: give each entry a tag for the stage that will close it and a status, so nothing
is silently dropped as the design moves downstream.

Tag each entry as one of:
- **behavior** — what the design should do; settled now with the designer (settling it changes
  the spec);
- **reference-defect** — a fault in the source model; fix or knowingly accept the reference;
- **hardware-binding** — a representational choice deliberately deferred to a later hardware
  stage (a width, an interface form, a table encoding);
- **must-define-for-hardware** — behavior left undefined that hardware cannot leave undefined
  (an illegal input, an error path, an out-of-range value).

Mark each entry `open`, `resolved`, or `deferred` (with a reason). A downstream stage inherits
the entries tagged for it and must close them before it is done.
