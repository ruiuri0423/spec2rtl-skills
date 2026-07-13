# hardware-spec

A Claude Code skill that turns a **functional specification** — the kind produced by the
`spec-recovery` skill — plus the **designer's hardware requirements** into a **hardware
architecture specification**: it resolves the hardware choices the functional spec deliberately
left open, introduces the hardware-specific structure a requirement demands (parallelism,
folding, feedback storage, interfaces), partitions the design into hardware blocks, and stops
there — short of RTL and micro-architecture.

*(This README was itself recovered from the skill by the `spec-recovery` method — reading
`SKILL.md`, stating its purpose, inputs, outputs, and behavior in plain language, and marking
what is not yet built.)*

## Where it sits

`hardware-spec` is the stage **after** functional understanding and **before** RTL:

```
spec-recovery  →  hardware-spec  →  (rtl-gen)
  what the        the hardware        the RTL
  design does     shape + why         itself
```

`spec-recovery` recovers *what a design does* and hands forward an open-questions ledger whose
`hardware-binding` and `must-define-for-hardware` entries are exactly this stage's inbox.
`hardware-spec` decides *what shape the hardware takes and why*, and hands the micro-architecture
forward to RTL generation.

## The core idea

The information that decides the hardware is **not in the code**. Throughput, clock, area,
interface, and how a block sits inside a larger system come from the **designer**, not the
algorithm. So this stage is **driven top-down by requirements, elicited from the designer**, and
it pushes those requirements down into each block's architecture.

The default hardware *is the functional specification mapped directly*. For a feed-forward design
with little feedback or crossing, the hardware simply follows the software's structure — and it
should. The skill **departs from that direct mapping only where a requirement forces it**, and it
names the requirement behind every departure, so the designer can hold the whole architecture in
their head and see why each piece is there. It introduces exactly the hardware the requirements
demand and no more.

## What goes in, what comes out

**Reads:** the functional specification (per-block specs and the top-level spec) and, above all,
its open-questions ledger; each block's *Resources* section (what it holds, what it computes, the
ranges its values take) as the raw material for sizing. What the spec cannot supply — throughput,
clock, latency, area, power, interfaces, integration context — it **elicits from the designer**.

**Delivers:** one hardware architecture specification, in prose, that shows the shape of the
hardware and the reason for it. For each block it states the architecture stance (follow the
software, or the specific departure and its driving requirement), the resolved bindings (widths,
interfaces), the storage the design holds, and how the whole design partitions into hardware
blocks. It carries its own ledger, now deferring the micro-architectural choices to RTL.

## How it works

1. **Elicit the requirements** — a fixed checklist (throughput, clock, latency, area or target
   device, power, the interfaces in and out, integration context), asked one convergent question
   at a time, each answer marked a **hard constraint** or a **target**.
2. **Set each block's stance, defaulting to follow-software.** A block that meets every
   requirement as-is is a complete answer, not an unfinished one.
3. **Apply the transformation a requirement demands** — parallelize/unfold for throughput,
   fold/share for area, add storage for state and feedback, define a protocol at each interface.
   The feedback case is the one with no software counterpart: find the loops, estimate the
   **iteration bound**, and use it as the floor on how far folding or pipelining can push the
   clock — a recursive loop cannot be sped past it.
4. **Partition into hardware blocks** — which need not match the software blocks (fuse trivial
   ones, split a block that must be parallelized, group by clock domain or interface).
5. **Record the architecture and its rationale**, deferring the micro-architecture to RTL.

## What it does not do

It does not write RTL and does not choose the micro-architecture — pipeline depth, scheduling,
register placement, FSM encoding — which belong to the RTL stage. It does not re-derive the
functional spec (it reads it). And it adds no architecture that no requirement asks for.

## Using it

Install as a Claude Code skill by placing this folder under a skills directory — per project at
`<project>/.claude/skills/hardware-spec/`, or for every project at `~/.claude/skills/hardware-spec/`
— then invoke `/hardware-spec` once a functional spec exists.

## Status

Draft. `SKILL.md` is complete; the reference material it points to is still to be written:

- `references/requirements-checklist.md` — the fixed requirement set, its convergent-card phrasing
  and defaults, and which architectural transformation each requirement can trigger.
- `references/method.md` — the per-block architecture reasoning: the follow-software default, the
  requirement→transformation analyses, and the iteration-bound / folding / feedback-storage work.
- `references/spec-template.md` — the structure of the delivered hardware architecture spec.
