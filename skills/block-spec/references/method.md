# method — how block-spec decomposes and budgets a block

This is the reasoning procedure the stage follows for each hardware block: inherit the block's
envelope from the hardware spec, allocate it across sub-modules as explicit budgets, decompose and
contract the sub-modules, and attach to each the evidence that will prove it. The budget algebra
says what to compute; this document is how the allocations are made, how the decomposition is
chosen, and the obligations that keep every number checkable.

## Working mode

The same obligations as the earlier stages, now bearing on allocations:

1. **Allocate in the open.** Every budget is derived by arithmetic that is written out; a split
   whose numbers are not shown is not an allocation, it is a wish.
2. **Budgets must sum.** Sub-module allocations sum to the block's envelope, with slack stated;
   stage counts along a path sum to the block's latency. A sum that does not close is a real
   finding to route, never something to round away.
3. **Show the source.** Every inherited figure names where it came from — which hardware-spec
   `Parameters:` line, which requirement, which functional-spec rule.
4. **Flag, don't fabricate.** A budget that cannot be evidenced — no delay model yet, no probe run
   — is marked with its assumption, or left open; the contract says so plainly.

The boundary obligation continues unchanged: this stage closes only the entries deferred to it —
the structural micro-architecture and the carried `must-define-for-hardware` items. A behavioral
question routes up to the functional spec's ledger; an architectural conflict routes up to the
hardware spec's; both via the ledger bus, never decided here.

## The budget algebra (the recipes)

Four budgets, allocated top-down from figures the hardware spec already derived:

- **Timing.** The clock period `T_clk` (from the requirements) bounds every pipeline stage. For
  each sub-module, state the combinational depth it may spend per stage — in ns, or in gate levels
  with the assumed per-level delay named as an assumption — and the pipeline stages `S` it is
  granted. A structure whose depth cannot fit the period at `S = 1` either pipelines (raising `S`,
  spending latency) or restructures (a linear scan becomes a binary search) — and the relation that
  decides is written out, because this is exactly where an unexamined structure fails timing at
  integration. The `S` along a path sum to the block's latency; check that sum against the latency
  requirement and any buffer the latency sizes.
- **Throughput.** The block's initiation interval (from the hardware spec's throughput relation) is
  inherited by every sub-module on the data path: each must accept one input per `II` cycles, and
  the rule by which sub-modules compose — how data advances from one to the next — must be stated
  so that composition provably preserves `II`, since bottom-up integration stands on it. That rule
  is not this skill's to choose and this skill names no default: it follows from the streaming
  discipline the requirements pinned (a stream that may not be back-pressured admits one kind of
  composition; an elastic interface admits others), and where the requirements did not pin it, ask
  the designer or record the assumption as an assumption.
- **Area.** The block's envelope — the multiplier count, the memory bits, the fold factor the
  hardware spec derived — is split across sub-modules as a table whose column sums to the envelope.
  Where a time-share schedule is what makes a split feasible, that schedule (which operations share
  which resource in which cycles) is stated here: it is a budget-bearing structure, not a coding
  detail.
- **State.** Every persistent value the hardware spec assigned to the block is owned by exactly one
  sub-module, and a feedback loop is never cut by a sub-module boundary unless the loop's timing
  budget is explicitly split across it — the iteration bound `T∞` travels with the loop.

## Decomposing into sub-modules

A **sub-module** is the unit one engineer — or one agent — can implement and verify as a whole. The
decomposition follows the budgets and the functions, never convenience:

- one coherent function from the specification — a contract a reader can state in a sentence;
- one budget of each kind it participates in;
- one verification obligation that can actually be discharged at its boundary;
- its state confined inside, its interfaces pinned.

Too big to verify as one piece — split it. Too small to carry a budget or be proven on its own —
merge it into its neighbor. Where the hardware spec's block already is one such unit, say so and do
not subdivide: no structure no requirement asks for.

## The contract, and its Budget line

Each sub-module's contract (the template gives the skeleton) opens with one checkable line:

> **Budget:** `II = … (from block §…)`, `T_comb ≤ … per stage over S = … stages (allocation §…)`,
> `A ≤ … (share of block envelope §…, table shown)`, `state = … (owned here)`.

This line is to this stage what `Parameters:` is to hardware-spec: the verify pass re-derives it,
and a contract without it is not a specification, it is a hope. After it come the **function** —
restating the exact numeric rules (rounding, clamping, scaling, sign handling) carried whole from
the functional spec, because a contract that paraphrases where it should restate gets implemented
wrong; the **interface**, with widths from the hardware spec's bindings and the advance rule the
streaming discipline dictates; the **state and reset**; and the **verification obligation** below.

## Verification obligations — named now, not after the code

Every contract names the evidence that will prove its implementation, before any implementation
exists:

- **Exhaustive golden**, where the input space permits — state the bound (a 13-bit space is
  exhaustible; a 48-bit one is not) and the reference that generates the golden.
- **Reference-dump golden** for composed paths — the reference model's per-stage dumps are the
  fixture; name which files prove which boundary.
- **Independent-implementation equivalence** as optional, stronger evidence for the hot spots — a
  second, structurally different implementation and a machine proof of equivalence, with the
  technique stated so any harness can reproduce it.

The pass criterion is the fidelity stance the functional spec settled. An obligation that cannot be
discharged at the sub-module's boundary is a sign the decomposition is wrong — fix the
decomposition, not the obligation.

## Tools are instruments, not the flow

A synthesis probe of a candidate structure is legitimate *evidence* for a timing or area budget —
record the tool, the flow, and the number it produced, as with any computed relation. Optimization
techniques found in generator/DSL toolchains — arithmetic restructuring, resource-sharing patterns,
equivalence checking — may inform a decision: state the technique itself, in the document, so the
decision stands without the tool. What the stage may never do is put a tool into the deliverable's
chain of custody: the flow delivers documents and source a team maintains, and an artifact only a
tool can regenerate — or a decision whose only justification is that a tool produced it — fails the
self-sufficiency test the same way a deleted draft does.

## The schedule

For a whole design, declared per the `pipeline` skill: **decompose and budget each hardware block**
*(independent tasks — one draft per hardware block: the allocation, the sub-modules, their
contracts)* → **verify each block** *(independent — one fresh re-derivation per drafted block)* →
**aggregate and converge** *(barrier — check the block-boundary interfaces agree, assemble the one
document with its merged ledger, route every surfaced question to its owner)*. Genuine PPA trades
go to the designer during drafting, one convergent question at a time with a recommended answer.
If the harness cannot iterate, budget the one block in hand and say which schedule tasks remain.

## The verify pass

A separate pass, per drafted block: re-derive every `Budget:` line from the hardware spec's
figures; re-sum every allocation table; re-run the relation behind every closed inbox entry and
confirm it was computed, not invoked; check that every verification obligation is dischargeable at
its boundary; and check the boundary obligation — nothing closed outside this stage's tags. At the
barrier, the form check runs once more over the assembled document: allocations, arithmetic, and
contracts must survive aggregation whole, per the library's carry-substance rule.

## What remains the model's judgment

Pre-RTL delay and area figures are estimates — per-level delays, operator costs — and what counts
as one sub-module is a judgment. The obligations make that judgment visible: every estimate is
named as one, every probe recorded with its flow, so that where judgment is uncertain it is
flagged, and the verify pass — and eventually the RTL stage's real measurements — are what correct
it. The stage does not pretend to precision it cannot have; it guarantees that every number can be
traced, re-derived, and later checked against the real thing.
