# requirements-checklist — the boundary, asked port by port

The information that decides the hardware is not in the code — and it is not abstract either: it
lives on the **boundary** of the hardware frame, as concrete properties of concrete ports. So
Phase A does not ask "what is the throughput?" into the air; it **enumerates the frame's inputs
and outputs** — data in, data out, configuration, status, clocks, resets, starting from the
functional spec's hardware-ization scope — and asks, per port, the property sheet below. Asked
this way, the elicitation is **complete by construction** (every port gets every applicable
property; no single item is "the highlighted question") and it does not anchor: a port's width,
discipline, or clock owner is a fact the designer owns directly, needing no framing from the
reference. Record every answer as a **hard constraint** or a **target**; many close a
`hardware-binding` entry in the functional ledger — note which as you go.

Three kinds of question take three forms, and this file serves the first. **Facts** about the
boundary are gathered by **enumeration** — the full port list and property sheet, visible at once;
a fact the designer cannot supply is OPEN, and a category's default applies only after an explicit
"unconstrained", never in place of an answer. **Evidence** questions (what the reference means)
belong to spec-recovery and converge one at a time with a recommended reading. **Trades** (two
defensible stances) are adjudicated one at a time in Phase B, both accounts at equal rigor.
Reference-implied figures enter only at the **reconcile step**, after the designer has answered:
computed from the functional spec, labeled with provenance, and diffed against the answers ("you
said X; the reference implies Y — is the difference intended?") — a cross-check, never an anchor.

## The per-port property sheet (Phase A)

For every enumerated port or port-group, elicit:

- **What it serves** — the functional-spec part this port realizes or configures (traceability).
- **Direction, width per beat, and the unit a beat carries** — a transport width is not the
  computation domain; the conversion between them resolves from the reference model's own
  convention, cited by line.
- **Rate and discipline** — the instantaneous rate that binds (formation/burst, never average), and
  whether the port may exert or suffer **backpressure** — asked, never defaulted.
- **Clock ownership** — which clock the port belongs to, by numeric frequency and owner.
- **Update discipline** (configuration ports) — when values may change relative to the data stream:
  shadowing, commit points.
- **Hard or target**, marked per property.

Two frame-level items no port carries — **area** and **power** — are asked once, as properties of
the frame; **latency** is the frame's in-to-out property, asked alongside the data ports. The seven
categories below remain as the **property definitions and their convergence relations**: Phase A
elicits each property on the boundary; Phase B fires the relation when quantifying the architecture
against it.

## How this works across models

This stage **drives a model's reasoning** with concrete convergence relations; it does not
prescribe the shape of the output. Each requirement below carries **the relation that decides it** —
a formula where one exists, a logical rule otherwise — that turns the requirement and the functional
spec's own numbers (from its Resources section) into a decision. Reason the relation, then state the
result and the number it gave. There are no rules here on length or wording, and none are needed:
the invariants are that every architectural choice **follows from a relation, not from style**, and
that it is stated **truthfully**. That is what keeps different models converging on the same
decision, and what lets the result hand forward to the RTL stage as justified numbers rather than
description.

Three habits keep this honest. First, ask only what you need: if a block plainly meets a requirement
as a direct mapping of its functional spec, the relation returns "no departure," and that is a
complete result. Second, wherever a relation forces a departure from the software's structure, name
the requirement in the spec beside the departure, so cause sits next to effect. Third — for **every**
requirement, clock and interfaces no more or less than area and power — **read the answer back
before you pin it**: restate the designer's answer in your own words, as an interpretation with its
architectural consequences made explicit ("I take this to mean the data path crosses two clock
domains, so CDC sits on the data path itself"), and let the designer confirm or correct the
restatement before it is recorded as a constraint. What the designer said and what the agent
understood are not the same thing, and the gap between them — an invented clock domain, an assumed
elastic interface — becomes architecture the moment it is written down as HARD. The read-back is
where such inventions die cheaply, one sentence early instead of one stage late. Two rules make a
read-back worth its name: **state numbers, units, and owners** — a clock is read back with its
numeric frequency *and what it clocks*, a rate with the unit of work it counts — and **never anchor
on figures from a superseded document**: restate from the designer's words alone, or the old number
quietly survives the correction wearing a new label.

Each requirement below is written as *what to ask*, *a default*, *constraint or target*, and *the
relation that decides it*.

---

## 1. Throughput — how fast must results come out?

- **Ask:** at what rate must the block accept inputs and produce outputs — samples, blocks, or
  results per second? Is that rate steady or bursty?
- **Default:** one result at a time — process an item, finish, take the next — no parallelism.
- **Usually:** a hard constraint, because the system feeds or drains data at a fixed rate.
- **The rate that binds is instantaneous, never average.** Under a streaming discipline that forbids
  backpressure, `rate` is the **formation rate of the unit of work during a burst** — how many units
  become computable per cycle while the stream delivers — not the long-term average. An average
  smuggles in the elastic buffer the discipline forbids. Note too that the unit of work may not be
  the interface's delivery unit (a beat of adjacent pixels is not a mirror-pair); when they differ,
  the formation rate comes from the formation *schedule*, derived in `method.md`.
- **The relation that decides it.** Let `C` be the block's natural cost in cycles per result (from
  its functional-spec Resources). Let the required initiation interval be `II = f_clk / rate` — the
  cycles you may spend per result (it may be < 1, meaning more than one result per cycle).
  - `II ≥ C` → the direct mapping already meets it; **no departure** (you may fold up to `⌊II/C⌋`, §4).
  - `1 ≤ II < C` → **pipeline** the datapath to an initiation interval of `II` (insert registers so a
    new item starts every `II` cycles).
  - `II < 1` → **replicate / unfold** the datapath by `⌈1/II⌉` and pipeline each copy.
  - If a **feedback loop** lies on the path, its iteration bound `T∞` (§2) is a floor on `II`: when
    `II < T∞`, pipelining cannot reach it — **unfold the loop by `⌈T∞/II⌉`**, or accept the bound.

## 2. Clock — what frequency does it run at?

- **Ask:** is there a target or fixed clock? Is it shared with the surrounding system, or free for
  this block to choose? And for **every named frequency: what does it clock** — which interfaces,
  which logic? A frequency without an owner is an unfinished answer, and the gap gets filled by
  invention (a "video clock" read back without its number and its owner became a fabricated compute
  domain once).
- **Default:** left open, set with the target technology once throughput and the iteration bound are
  known.
- **Either:** hard when the block sits in a fixed clock domain; a target when it may pick its own.
- **The relation that decides it.** With clock period `T_clk = 1/f` and combinational critical-path
  delay `t_cp`, the feed-forward path needs **`S = ⌈t_cp / T_clk⌉` pipeline stages** to close timing.
  A **feedback loop cannot be pipelined across**: its **iteration bound** `T∞ = max over loops (loop
  computation delay ÷ number of registers in the loop)` is the smallest achievable iteration period.
  So if the target period is shorter than `T∞` allows for a recursive path, no amount of pipelining
  helps — the loop must be **unfolded, retimed, or restructured (look-ahead)**, or the clock target
  on that path lowered.

## 3. Latency — how long from input to output?

- **Ask:** is there a limit on the delay from an input arriving to its result appearing — a
  real-time deadline, or a cycle budget? Per item, or end to end?
- **Default:** whatever the natural dataflow gives; no explicit ceiling.
- **Either:** hard under a real-time deadline; otherwise a target.
- **The relation that decides it.** Total latency `L_tot ≈ (pipeline depth `S` + folding passes) ×
  T_clk` must satisfy `L_tot ≤ L_budget`. This **bounds the §1 and §4 choices from the other side**:
  `S ≤ L_budget / T_clk` limits how deep you may pipeline for clock, and folding (which multiplies
  latency by its factor `F`) is limited to `F ≤ L_budget / (C · T_clk)`. Where throughput wants deep
  pipelining or area wants heavy folding and latency forbids it, the spec records the trade and where
  it landed.

## 4. Area — how much silicon may it spend?

- **Ask:** is there a resource or area budget, or a target device (an FPGA part, an ASIC node)? Is
  the block one of many sharing it?
- **Default:** minimize, with no hard ceiling.
- **Either:** hard when it must fit a device; a target otherwise.
- **The relation that decides it.** With natural (fully parallel) resource cost `A_nat` and budget
  `A_bud`, the **fold factor is `F = ⌈A_nat / A_bud⌉`** — reuse one datapath across `F` computations,
  which divides throughput by `F` and multiplies latency by `F`. The fold is valid only if it still
  meets the other requirements: **`natural_rate / F ≥ required_rate` (§1)** and **the folded
  iteration period stays `≥ T∞` on any recursive path** (a feedback loop cannot be folded below its
  bound). If `F` from area would violate §1, the area target cannot be met at this throughput — a
  trade to record, not to hide.

## 5. Power — is there a power budget?

- **Ask:** is there a power or energy limit, or a low-power target? Is the block mostly idle or
  always active?
- **Default:** no explicit budget; power follows from the area and clock choices.
- **Usually:** a target rather than a hard constraint.
- **The relation that decides it (mostly derived).** Dynamic power `P ∝ α · C_load · V² · f` — it has
  no datapath formula of its own and is driven by activity `α`, switched capacitance `C_load`
  (≈ area), voltage, and frequency. So unless it is a hard budget, **close it through §2 (lower `f`)
  and §4 (smaller, folded → less `C_load`)**, and gate the idle regions (activity `α`). If it is a
  hard budget, it further tightens the `f` and `A` a design may use.

## 6. Interfaces — how does data enter and leave, and how is it controlled?

- **Ask:** for each boundary — data in, data out, control/status — how does it connect (a streaming
  interface, a memory-mapped port, a register file)? Who initiates the transfer? **May the design
  exert backpressure on its input, and may its output stall — or must the stream flow
  unconditionally?** This is a fork that shapes everything downstream (elastic buffering versus
  static scheduling) and it is **asked, never defaulted**. And for each direction: the width per
  beat and the width per datum (a transport width is not the computation domain — §how the two map
  is part of the answer).
- **Default:** a small register interface for control and status. The data-side discipline
  (unconditional stream versus handshake) has **no default** — it is the designer's fork.
- **Usually:** hard constraints, because they must match the surrounding system exactly.
- **The relation that decides it (logical, with a numeric part).** The protocol at each boundary must
  **match the peer exactly** — this is where the functional spec's deferred "what form does the input
  take" bindings are finally closed. Two mismatches add hardware by rule. Rate mismatch: where the
  discipline **permits backpressure**, insert a **FIFO** of depth `≥ B · (1 − r_c / r_p)` (producer
  burst `B`, producer rate `r_p` > consumer rate `r_c`); where it **forbids** backpressure, the
  mismatch is closed by a **statically scheduled delay line** — its schedule derived and simulated
  per `method.md`, never an elastic element. Width mismatch: **pack / unpack** logic at the boundary,
  with the conversion into the computation domain resolved from the reference model's own convention
  (cited by line), because the model is authoritative on the domain arithmetic runs in.

## 7. Integration — where does this block live?

- **Ask:** is the block standalone or part of a larger IP/SoC? One clock domain or several? Who
  drives it — master or slave? What is the reset scheme?
- **Default:** standalone, a single clock domain, externally driven, synchronous reset.
- **Usually:** a hard constraint dictated by the system it joins.
- **The relation that decides it (logical, structural).** With `D` clock domains, **every data path
  that crosses a domain boundary needs a crossing element** — an asynchronous FIFO for data, a
  synchronizer for control — so the number of CDC elements follows the number of crossings. A
  master/slave role and a control/status surface fix the **register map**. And the integration shape
  drives the **partition**: group logic by clock domain and by interface (this is why partition lives
  in this stage, not a separate one), and propagate the reset scheme to every block.

---

## Hard constraint versus target

Mark every answer. A **hard constraint** must be met and bounds the design space — a design that
misses it is invalid. A **target** is approached and traded against the others; when two pull apart
(throughput up, area up; latency down, throughput down), the relations above make the tension
explicit, and the spec records where it landed and why. Keeping the two apart is what lets a later
reader tell a decision that *had* to be made from one that was *chosen*.

## How these shape the architecture

Read together, the relations are the pressure that bends a design away from the plain software
mapping. Where no relation forces a departure, the block stays a direct mapping and the spec says so.
Where one does, exactly one transformation answers it, named beside the requirement that forced it,
with the number the relation gave. The worked reasoning for each transformation — how to find `C`,
`A_nat`, and the loops that set `T∞`, and how the transformations compose — lives in `method.md`.
