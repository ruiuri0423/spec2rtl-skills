# method — how hardware-spec reasons about a block

This is the reasoning procedure the stage follows for each block: obtain the parameters the
convergence relations need, run the relations from `requirements-checklist.md`, and compose the
transformations into an architecture. The relations tell you *what* to compute; this document is
*how* to obtain their inputs, *how* the transformations combine, and — most important for a result
that survives being handed forward — the obligations that make every number checkable.

## The working mode, made concrete

The stage drives a model's reasoning; it does not prescribe the shape of the output. A skill cannot
give a model reasoning it does not have — so the goal is not that every model produce the same
answer, but that every model produce a **true and verifiable** one, degrading to honest open
questions where it cannot. That is enforced not by rules on form but by four obligations on every
parameter and decision:

1. **Follow the recipe, don't eyeball.** Obtain each parameter (`C`, `A_nat`, the loops that set
   `T∞`) by the checkable procedure below, not by impression.
2. **Compute in the open — and simulate what arithmetic cannot settle.** Evaluate a relation with a
   tool where the agent has one; otherwise write the arithmetic out step by step. Never hide it in
   mental math. A claim about a *schedule* — when units of work form, whether buffer access windows
   collide, where an occupancy peaks — is beyond inspection: verify it by **cycle-accurate
   simulation** (a short script over the arrival/consumption equations), and carry the result into
   the document like any computed number. An off-by-one in a schedule is a real hardware bug; eyes
   do not catch it, simulation does.
3. **Show the derivation and its source.** State each parameter with how it was derived and where it
   came from — which functional-spec section, or which elicited requirement.
4. **Flag, don't fabricate.** If a parameter cannot be obtained reliably, mark it `UNCONFIRMED` or
   ask the designer. The relation then does not fire and the block's stance stays open — a truthful
   result, not a failure. This is how the stage stays honest on a weaker model: it says less, never
   something false.

These are obligations of accountability, not of style. They constrain whether a claim can be
checked, never how a model reasons or how much it writes.

One further obligation governs the stage's **boundary**. This stage may close only the ledger
entries tagged for it — `hardware-binding` and `must-define-for-hardware`. When the hardware work
uncovers a question outside those tags — a *behavioral* ambiguity the functional spec never pinned,
an interface whose meaning is unclear, a suspected defect in the reference — do not quietly decide
it here, however obvious the answer seems. Route it back into the functional spec's ledger (tagged
`behavior` or `reference-defect`, status `open`) for the functional stage and the designer to
settle, then re-read what this stage depends on. The `pipeline` skill defines this routing; the
obligation here is simply: **never resolve what you are not tagged to own.**

Two authorities flank the stage, and each is final on its own ground. **The designer** owns the
requirements and the trades — and may overturn a drafted stance mid-stage: a designer challenge
("your buffering costs more than the multipliers it saves") triggers **re-derivation at the
challenge's level of rigor**, never a defense of the draft; if the re-derivation agrees, the stance
flips and the designer's decision is recorded with a date. Facts the designer pins from experience
— a memory organization, a word width — are recorded as pinned, with the designer named. **The
reference model** owns the computation: the domain arithmetic runs in, and every conversion into
that domain, resolve from the model's own conventions, cited by line — a transport width the
designer supplies never redefines the domain the model computes in.

## The schedule

For a whole design, this stage declares a schedule and leaves how it runs to the `pipeline` skill —
stepwise or concurrent, the outputs are identical: **elicit the requirements** (one convergent
conversation; its answers feed everything) → **architect each block** *(independent tasks — one per
block: parameters, relations, stance — each producing a draft, not a deliverable)* → **verify each
block** *(independent — one fresh derivation per drafted block)* → **partition, aggregate, and
converge** *(barrier — needs every verified block: the hardware partition, the interfaces between
blocks, and the assembly of the drafts into the one architecture document — one body, one merged
ledger, one Revision log — with every surfaced question routed to its owner)*. The deliverable is
that single document; per-block drafts do not survive the barrier, so every later resolution —
including an answer routed back from the functional stage — lands in exactly one place (the
`pipeline` skill defines this aggregation point). And because the drafts do not survive, **the
assembly carries each block's substance into its section whole** — the `Parameters:` line *and* the
open arithmetic behind every cited relation (the enumerated `C`, the operation account behind
`A_nat`, the relation computed out to its number). Assembly reconciles and deduplicates; it never
summarizes: a section that states a stance and points at a draft for the derivation has destroyed
the evidence the obligations exist to preserve, and the downstream stages inherit an impression
they cannot check. If the harness cannot iterate over files, architect the one block in hand and say which
schedule tasks remain.

## Obtaining the parameters (the recipes)

Three parameters come from the functional spec; the rest come from the elicited requirements.
State them as one checkable line per block — `Parameters: C = … (from …), A_nat = … (from …),
feedback loops / T∞ = … (or none)` — before any stance is argued; the spec-template requires this
line, and the verify pass re-derives it. A stance whose parameters were never stated is not an
architecture, it is an impression.

- **`C` — natural cost, cycles per result.** From the block's Behavior and Resources: enumerate the
  steps the algorithm takes to produce one result, in order, and sum their cycle costs. Show the
  enumeration. *(Example: a zig-zag decoder that visits 64 grid positions, one write per position,
  has `C ≈ 64` — because the Behavior section describes a 64-step walk.)*
- **`A_nat` — natural, fully-parallel resource cost.** From Resources' account of the work per
  result: list the operations the block performs — multiplies, adds, compares, memory words — as if
  all were done at once, and total them. Show the list. This is a relative measure for sizing the
  fold factor, not a gate count.
- **Feedback loops and `T∞`.** From the functional spec's *persistent state* and its cross-call /
  cross-block dependencies: every value carried from one result to the next, or any output that
  becomes a later input, is a feedback loop. List them. For each, `T∞ = loop computation delay ÷
  registers in the loop`, and the block's iteration bound is the maximum over its loops. If Resources
  records no persistent state and no cross-result dependency, say so plainly: **no feedback → no
  iteration bound**, and every path may be pipelined and folded freely. *(Example: JPEG's DC
  predictor — each block's DC adds the previous block's DC — is a one-register loop around an adder;
  its `T∞` is that adder's delay, and it caps how fast the DC path can go.)*
- **`f_clk`, `rate`, `A_bud`, `L_budget`, interface rates and widths** come from the requirements
  (elicited), not the spec — mark each as the hard constraint or target it was recorded as.
- **The unit of work, and its formation schedule.** The algorithm's unit of work is often not the
  interface's delivery unit (a beat of adjacent same-view pixels is not a mirror-pair). When they
  differ, derive the **formation schedule** before any throughput relation fires: from the delivery
  order, when does each unit become computable, and at what instantaneous rate do units form during
  a burst? That formation rate — not the interface rate, not the long-term average — is the `rate`
  the §1 relation consumes; and the schedule itself (with the buffering it implies) is part of the
  architecture, verified by cycle-accurate simulation like any schedule claim.
- **Storage, stated as physical organization.** Any memory the architecture claims is specified as
  macros — ports, banks, word width, and what one word *is* (usually one interface beat) — plus the
  domain of the stored values (raw transport-width data before conversion stores narrower than the
  computation domain). **Never budget storage at live-data peaks**: two buffers of equal peak
  occupancy differ by 2–3× physically once same-cycle access windows force ping-pong or a second
  port, so prove the windows disjoint (simulate them) or pay for the organization that survives the
  collision. This accounting decided an engine-rate stance once — at live-data peaks one plan looked
  smaller; at physical organization the other was a third the size.

## Composing the transformations

Start every block from the **direct mapping** of its functional spec, then apply the relations in an
order that respects how they interact — because they pull against each other and against the
iteration bound:

1. **Throughput** first sets whether the natural rate suffices, and if not, the pipeline or unfold
   factor (§1 of the checklist).
2. **Area** may then fold to fit the budget — but folding divides throughput, so the fold factor is
   bounded by the throughput requirement, and on any recursive path by `T∞` (§4). A fold that would
   break §1, or push a loop below its bound, is not available.
3. **Latency** caps both from the other side: deep pipelining and heavy folding each add latency, so
   a tight budget limits how far either may go (§3).
4. **The iteration bound `T∞` is the hard floor** beneath all of this on any feedback path — no
   pipelining, folding, or clock target can cross it; only restructuring the algorithm (unfolding,
   look-ahead, retiming) can.

If the constraints are jointly infeasible — the throughput needs more than the area allows, or the
clock target is below a loop's bound — that is a real finding to report as a trade, with the numbers
that show the conflict. It is never something to fudge into looking solved.

When two stances genuinely compete — two schedules, two engine rates, two structures that both meet
the requirements — **the loser is rejected only on accounting of the same rigor as the winner's**,
and that accounting is recorded beside the rejection. A rejection argued on a shallower ledger than
the adoption (live-data bits against physical macros, average rate against burst rate) is not a
decision, it is a coin toss that happened to land — and it is exactly the kind the designer will
later overturn. Presenting both accounts at equal depth is also what makes the trade *challengeable*,
which is the point: the designer decides trades, and can only decide what they can check.

Record each resulting choice beside the requirement whose relation forced it, with the number the
relation produced. Where no relation forced a departure, the block stays a direct mapping and the
spec says so.

## The verify pass

A block's architecture is not passed forward on the strength of one derivation. After it is drafted,
a **separate pass re-derives its parameters and re-runs its relations independently**, and flags any
mismatch — a different `C`, a feedback loop the first pass missed, an arithmetic slip, or a
transformation whose stated driving requirement does not actually force it. Only a block whose
parameters and decisions survive re-derivation is passed on; a mismatch is surfaced, not smoothed.

The verify pass checks form as well as values. In particular: **was each cited relation actually
computed?** A stance that *invokes* a relation ("II < C, therefore pipeline") without its
`Parameters:` line and the arithmetic is a finding — the most common way a derivation quietly
degrades into an impression. Three further checks, each bought by a real failure: **was every
schedule claim simulated** (not inspected — an unsimulated schedule once shipped an off-by-one in
the constant a delay pipe is built to)? **Is every storage claim stated as physical organization**
(a live-data-peak budget is a finding)? **Was every rejected alternative rejected at the winner's
rigor** (a shallow rejection is re-opened, not accepted)? It also checks the boundary obligation:
any entry resolved here that carries a tag this stage does not own is flagged for re-routing, not
accepted. And the same form check runs once more over the **assembled document** at the barrier —
assembly is where derivations are most easily lost, so a block section whose stance survived but
whose arithmetic did not is a finding against the assembly, not the draft.

Because this re-derivation runs as its own agent — and, for the load-bearing numbers, can run more
than once — the guarantee does not rest on any single model call being right. It rests on the
decision being **reproducible**. This is the system-level answer to models diverging: do not trust
one derivation of a parameter; verify it, and pass forward only what a fresh derivation confirms.

## What remains the model's judgment

Some steps are irreducibly a matter of judgment: what counts as "one operation" when totting up
`A_nat`, whether a dependency is a true feedback loop or merely a sequence, how to estimate a
combinational delay before any RTL exists. The obligations above do not pretend to remove this
judgment — they make it **visible and checkable** (shown with its source, re-derived by the verify
pass) so that where judgment is uncertain, it is flagged rather than hidden. A skill can make
reasoning accountable; it cannot manufacture the reasoning itself, and it should not pretend to.
