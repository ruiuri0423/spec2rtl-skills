---
name: block-spec
description: >
  Turn the hardware architecture specification (from hardware-spec) into per-block implementation
  specifications: decompose each hardware block into sub-modules, give every sub-module a
  functional contract and an explicit top-down PPA budget — timing, throughput, area, state — and
  pin the interfaces between them, so RTL can then be written and verified bottom-up, one
  sub-module at a time, against specs that already close. Use it as the stage after hardware-spec
  and before RTL generation.
---

# block-spec

An architecture specification says what shape the hardware takes and why. It does not yet say what
each piece must do, or what each piece may spend — and RTL written straight from an architecture
answers those questions implicitly, in code, where no one can check them until integration fails.
This stage writes the answers down first. It decomposes each hardware block into **sub-modules** —
the unit one engineer, or one agent, implements and verifies as a whole — and gives every
sub-module a **contract**: the function it computes, the interface it presents, the state it holds,
the budget it may spend, and the evidence that will prove it correct. Requirements flow **top-down**
as allocated budgets precisely so that implementation can then proceed **bottom-up**: each
sub-module provable against its own contract, in isolation, before anything is integrated.

The convictions of the earlier stages carry through unchanged. Every allocation follows from
arithmetic that is shown and must sum — a budget is a number with a derivation, not an aspiration.
The deliverable is one readable document, self-sufficient, with nothing pointing elsewhere for its
substance. And the stage must leave the design **maintainable by people**: what it hands forward is
specifications — and, downstream, source — that a team can own; never an artifact only a tool can
regenerate or explain.

## What this skill does

It apportions the design's requirements down the hierarchy — the design-to-block split was
hardware-spec's job; block-to-sub-module is this one's — as explicit budgets: the combinational
time a sub-module may spend inside the clock, the initiation interval it must sustain, the area
envelope (multipliers, memory bits) it may occupy, and the state it owns. It decomposes each block
into sub-modules along those budgets, writes each sub-module's functional contract (restating the
exact numeric rules the functional spec pinned), pins the interfaces between sub-modules, and
attaches to each contract a **verification obligation** — the golden evidence that will later prove
the implementation, named now, while the spec is written, not after the code exists. In doing so it
closes the hardware spec's *structural* micro-architecture entries — the choices that move budgets:
a pipeline depth, a search structure, a time-share schedule — each closed by a computed relation.

## What this skill reads

The hardware architecture specification — its requirements, each block's `Parameters:` line, its
stance and arithmetic, its resolved widths — and above all its **ledger**: the micro-architecture
entries deferred to this stage (the structural, budget-bearing choices) and the
`must-define-for-hardware` entries the hardware stage carried down. Where a contract must restate
behavior, it reads the functional specification's rules and carries them whole. What no spec can
supply — a genuine PPA trade with more than one defensible point — goes to the **designer**, one
convergent question at a time, each with a recommended answer and the numbers behind it.

## What this skill delivers

One block implementation specification — a single document, per the library's aggregation rule: a
budget-allocation overview whose sums are shown, then one chapter per hardware block and one
contract per sub-module, each contract opening with a checkable `Budget:` line. It carries its own
ledger, now deferring only the coding-level choices — FSM encoding, register placement inside a
stage, reset detail — to the RTL stage. What the document contains is set out in
`references/spec-template.md`.

## How it works

The full procedure is in `references/method.md`; in outline:

- **Inherit, then allocate.** Take each block's envelope from the hardware spec and split it across
  sub-modules with the arithmetic shown; every allocation must sum, slack is stated, and an
  allocation that cannot close is a trade routed to its owner, never fudged.
- **Decompose along budgets and functions.** A sub-module is one coherent function, one budget of
  each kind, one dischargeable verification obligation, its state confined inside; too big to
  verify as one piece → split, too small to carry a budget → merge.
- **Contract each sub-module** with a `Budget:` line the verify pass re-derives, the function
  restated whole, the interface pinned, the golden evidence named.
- **Verify by independent re-derivation** — a separate pass re-sums every allocation, re-runs every
  relation behind a closed entry, and form-checks the assembled document at the barrier.
- **Tools are instruments, not the flow.** A synthesis probe may supply evidence for a budget, and
  optimization techniques may be borrowed from generator toolchains — but the deliverables are
  documents and source a team maintains, and no decision may rest on "a tool produced it" without
  the technique itself being stated and checkable.

## What this skill must not do

It does not write RTL, and it does not make coding-level choices — FSM encoding, register placement
within a pipeline stage, reset implementation detail — which belong to the RTL stage that reads
this document. It does not re-derive the architecture: a departure it believes necessary is routed
back to the hardware spec's ledger, not decided here. It adds no sub-module and no budget that no
requirement asks for. And it never fabricates feasibility: where an allocation cannot be made to
sum, or a budget cannot be evidenced, the entry stays open with the conflict shown.
