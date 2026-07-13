---
name: hardware-spec
description: >
  Turn a functional specification (from spec-recovery) plus the designer's hardware
  requirements into a hardware architecture specification — resolving the deferred hardware
  bindings, introducing the hardware-specific architecture (parallelism, folding, feedback
  storage, interfaces) that a requirement demands, and partitioning the design into hardware
  blocks — while stopping short of RTL and micro-architecture. Use it as the stage after
  spec-recovery and before RTL generation, once a functional spec exists and the design is
  ready to be given a hardware shape.
---

# hardware-spec

A functional specification says what a design does, at the level of meaning, and deliberately
leaves the hardware open. This stage decides the hardware. But the information that decides it
is not in the functional spec, and it was never in the reference code either: the throughput a
block must sustain, the clock it runs at, the area it may spend, the interface it presents, and
the way it sits inside a larger system come from the **designer**, not from the algorithm. So
this stage is driven **top-down by requirements**, elicited from the designer, and it pushes
those requirements down into the architecture of each block.

The guiding idea is that the default hardware *is the functional specification mapped directly*.
For a feed-forward design with little feedback or crossing, the hardware can simply follow the
software's structure, and it should. This stage departs from that direct mapping only where a
requirement forces it, and it names the requirement behind every departure. It introduces
exactly the hardware structure the requirements demand and no more, so the designer can hold the
whole architecture in their head and see why each piece is there. And it stops at architecture —
what is parallelized, what is folded, what holds state, how data crosses each boundary, with the
reasoning — leaving the micro-architecture to the next stage.

## What this skill does

It elicits the hardware requirements the algorithm cannot supply, resolves the functional spec's
deferred hardware bindings and its undefined-for-hardware points against those requirements, and
works out the architecture each block needs to meet them — the parallelism, the folding, the
storage for feedback, the interfaces — together with how the design partitions into hardware
blocks. Every architectural decision is recorded with the requirement that drove it, so a reader
never finds a piece of hardware without knowing why it is there.

## What this skill reads

It reads the functional specification produced by spec-recovery — the per-block specs and the
top-level spec — and above all their **open-questions ledger**: the entries tagged
`hardware-binding` and `must-define-for-hardware` are precisely this stage's inbox, and closing
them is part of its job. Each block spec's Resources section — what the block must hold, what it
must compute, the ranges its values take — is the raw material for sizing widths and for deciding
what is worth parallelizing or folding. What the spec cannot give — the throughput, clock,
latency, area, power, interface, and integration context — this stage **elicits from the
designer**; that elicitation is its main input, not a footnote.

## What this skill delivers

One hardware architecture specification, written so a person can read it and see the shape of the
hardware and why it took that shape. For each block it states the architecture stance — follow
the software directly, or the specific departure and the requirement behind it — the resolved
bindings (widths, interfaces), the storage the design must hold, and how the whole design
partitions into hardware blocks. It carries its own open-questions ledger, now deferring the
micro-architectural choices to the RTL stage. What that document should contain is described in
full in `references/spec-template.md`.

## How it works

1. **Elicit the requirements.** Ask the designer the fixed requirement checklist
   (`references/requirements-checklist.md`) — throughput, clock, latency, area or target device,
   power, the interfaces in and out, and the integration context — one convergent question at a
   time, each carrying a sensible default, and mark every answer a **hard constraint** or a
   **target**. These answers both close the functional spec's `hardware-binding` questions and
   set the pressure that shapes the architecture.
2. **Set each block's stance, defaulting to follow-software.** Start every block from the direct
   hardware mapping of its functional spec. Then, for each requirement, ask whether it forces a
   departure — and only then depart. A block that meets every requirement as-is is specified as a
   direct mapping, and that is a complete answer, not an unfinished one.
3. **Apply the transformation the requirement demands.** Each hardware-specific change is tied to
   the requirement that triggers it (see `references/method.md` for the analysis of each):
   parallelize or unfold when throughput exceeds the block's natural rate; fold or share when the
   area budget is below its natural size; add storage for state and feedback; define a protocol
   at each interface. The feedback case is the one with no software counterpart — find the
   feedback loops from the functional spec's persistent state and cross-block dependencies,
   estimate the **iteration bound**, and use it as the floor on how far folding or pipelining can
   push the clock: a recursive loop cannot be sped past it.
4. **Partition into hardware blocks.** The hardware blocks need not match the software blocks:
   fuse trivial ones, split a block that must be parallelized, group by clock domain or interface.
   Because the partition follows from the transformations above, it is decided here, not in a
   separate stage.
5. **Record the architecture and its rationale**, and hand the micro-architectural choices forward
   as a ledger for the RTL stage to close.

## What this skill must not do

It does not write RTL, and it does not choose the micro-architecture — pipeline depth, scheduling,
register placement, FSM encoding — which belong to the RTL stage. It does not re-derive the
functional specification; it reads it. And it does not add architecture that no requirement asks
for: with no driving requirement, the hardware follows the software, and the specification says so
plainly.
