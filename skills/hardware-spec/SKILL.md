---
name: hardware-spec
description: >
  Turn a functional specification (from spec-recovery) plus the designer's hardware requirements
  into a hardware architecture specification — resolving the deferred hardware bindings, introducing
  only the parallelism, folding, feedback storage, and interfaces a requirement demands, and
  partitioning the design into hardware blocks — while stopping short of RTL and micro-architecture.
  Use it as the stage after spec-recovery and before RTL generation, once a functional spec exists
  and the design is ready to be given a hardware shape.
---

# hardware-spec

A functional specification says what a design does and deliberately leaves the hardware open. This
stage decides the hardware — but the information that decides it was never in the code: the
throughput a block must sustain, the clock it runs at, the area it may spend, the interface it
presents, and how it sits inside a larger system come from the **designer**, not the algorithm. So
this stage is driven **top-down by requirements, elicited from the designer**, and it pushes those
requirements down into the architecture of each block.

Two convictions carry over from spec-recovery and shape everything here. The first: the default
hardware *is the functional specification mapped directly*. For a feed-forward design with little
feedback or crossing, the hardware follows the software's structure, and it should; the stage
departs from that mapping only where a requirement forces it, names the requirement behind every
departure, and adds no structure no requirement asks for — so the designer can hold the whole
architecture in their head. The second: the stage **drives reasoning rather than dictating output**.
Each requirement carries a convergence relation — a formula where one exists, a logical rule
otherwise — and every architectural choice must *follow from a relation, shown and checkable*, not
from style. There are no rules on length or wording; the invariants are that decisions follow from
relations and are stated truthfully.

## What this skill does

It elicits the hardware requirements the algorithm cannot supply, resolves the functional spec's
deferred bindings and its undefined-for-hardware points against them, and works out the architecture
each block needs — the parallelism, the folding, the feedback storage, the interfaces — together
with how the design partitions into hardware blocks. Every decision is recorded beside the
requirement that drove it and the number its relation produced, so no piece of hardware appears
without its reason.

## What this skill reads

It reads the functional specification from spec-recovery — one document, with its top-level and
per-block sections — and above all its **open-questions ledger**: the entries tagged `hardware-binding` and
`must-define-for-hardware` are this stage's inbox to close. Each block's *Resources* section — what
it holds, what it computes, the ranges its values take — is the raw material for the parameters the
relations need. What the spec cannot supply — throughput, clock, latency, area, power, interfaces,
integration context — this stage **elicits from the designer**, and that elicitation is its main
input, not a footnote.

## What this skill delivers

One hardware architecture specification, in prose, that shows the shape of the hardware and the
reason for it: for each block, its architecture stance (a direct mapping, or the specific departure
and the requirement behind it), the resolved bindings, the storage it holds, and how the whole
design partitions into hardware blocks. It carries its own open-questions ledger, now deferring the
micro-architectural choices to the RTL stage. What the document should contain is set out in
`references/spec-template.md`.

## How it works

The requirement set is in `references/requirements-checklist.md`; the full reasoning procedure —
obtaining the parameters, composing the transformations, and verifying them — is in
`references/method.md`. In outline:

- **Elicit the requirements** — a fixed checklist (throughput, clock, latency, area, power,
  interfaces, integration), asked as a convergent conversation, each answer marked a hard constraint
  or a target.
- **Set each block's stance, defaulting to follow-software**, and depart only where a requirement's
  relation forces it — parallelize or fold, add feedback storage bounded by the iteration bound,
  define a protocol at each interface — naming the requirement and the number beside the departure.
- **Partition into hardware blocks**, which need not match the software blocks, following from the
  transformations above.
- **Show, compute in the open, and verify** — every block opens with a checkable `Parameters:` line
  (C, A_nat, T∞, each with its source); the arithmetic is done with a tool or written out; and a
  separate pass re-derives the parameters and confirms each cited relation was actually computed.
- **Stay inside the stage's authority** — this stage closes only the ledger entries tagged for it;
  a behavioral question uncovered here is routed back to the functional spec's ledger, never decided
  quietly (the `pipeline` skill defines the routing). How a whole design's schedule runs — stepwise or
  concurrent — is likewise governed by the `pipeline` skill.
- **Flag, don't fabricate** — a parameter that cannot be obtained is marked open, not guessed.

## What this skill must not do

It does not write RTL and does not choose the micro-architecture — pipeline depth, scheduling,
register placement, FSM encoding — which belong to the RTL stage. It does not re-derive the
functional spec; it reads it. It adds no architecture that no requirement asks for. And it does not
fabricate a parameter to make a relation fire: where the input cannot be obtained, the decision
stays open.
