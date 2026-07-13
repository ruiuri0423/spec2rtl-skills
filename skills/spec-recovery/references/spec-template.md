# Functional Specification — <block name>

This document is the output of the spec-recovery stage: what a piece of reference software does,
said plainly enough that someone who has never seen the code understands the design, and completely
enough that the later stages can build on it. It stops at *what the block does* — no clock, no
interface, no memory organization, no bit widths. Those are decided later, by the skills that read
this document; asking for them here would force a reader to commit to hardware before understanding
the design well enough to choose. This stage gathers requirements only, and hands them forward.

Write every section as prose — explaining not only *what* is true but *why* and *how*, the way you
would to a new colleague — and keep each section on its own point. The prose carries the design;
everything *about* the document — its status, what has changed, and what is still open — lives in the
dedicated fields (the status line, the open-questions ledger, and the Revision log at the foot),
never woven into the prose as an aside. Where the reference does not settle something, do not guess:
write `UNCONFIRMED:` and carry it to the ledger.

*Derived from: `<source file>`. Status: draft, pending the open questions below.*

---

## 1. Purpose — what this block is for

Why this block exists and what problem it solves for the system around it. Say where it sits — what
comes before, what comes after — and state the boundary of its responsibility plainly (for example,
"it produces quantized coefficients, not pixels"), because a reader's first mistake is almost always
about scope.

## 2. Inputs — what goes in

Everything the block consumes. For each input, what it represents, what form it takes, and any
convention that gives it meaning. Include the inputs that are not obvious from the interface — data
the code loads, tables it relies on, constants it quietly assumes — because a maintainer who does
not know these exist cannot use the block. If an input's form is only a convenience of the reference
model rather than its true nature, say so.

## 3. Outputs — what comes out

Everything the block produces, the same way. Watch for results the code computes and uses internally
but never hands back: if the surrounding system needs such a value, it is an output too, and
recording it here is what lets the block connect to whatever comes next.

## 4. Resources — what it needs to hold and to compute

At the level of the algorithm, what the block must *remember* and what work it must *do*. For memory:
which values hold state, how large, how indexed, and how long they must live — within one run, or
across runs. For computation: the significant operations and roughly how many, per input, element,
or block. This is not a hardware budget and names no gates or registers; it is an honest account of
the demands the computation makes, so a later stage can size real hardware from something concrete.

## 5. Behavior — what it does

How the block turns its inputs into its outputs. Walk the steps in order — the phases, the loops and
what ends them, the decisions and what drives them — and include the numeric rules inside them: how
signs are handled, how values are scaled or rounded, where a result depends on an earlier one. This
is the heart of the specification; write it so a reader could reconstruct the intended behavior
without ever seeing the original code.

## Open questions (ledger)

Every point the design still leaves open — what is unresolved, why it matters, and what would settle
it. This is a ledger that travels with the spec: give each entry a tag for the stage that will close
it and a status, so nothing is silently dropped downstream. Tag each entry `behavior` (what the
design should do — settled now with the designer), `reference-defect` (a fault in the source model —
fix or knowingly accept it), `hardware-binding` (a representational choice deferred to a later
hardware stage), or `must-define-for-hardware` (behavior left undefined that hardware cannot leave
undefined). Mark each `open`, `resolved`, or `deferred` (with a reason). A downstream stage inherits
the entries tagged for it and must close them.

## Revision log

Newest first, one line each: what changed in this document and why. The sections above always read
as the current truth; this log — not scars in the prose — is where the history lives. Add an entry
whenever the spec is updated. This is distinct from the ledger above: the ledger tracks the state of
design *decisions*; this log tracks changes to the *document*.

- `<date or version>` — <what changed, and why>.
