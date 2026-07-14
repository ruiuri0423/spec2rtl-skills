# Hardware Architecture Specification — <design or block name>

This document is the output of the hardware-spec stage: the shape the hardware takes, and the reason
for it. It reads a functional specification and the designer's requirements, and it decides the
architecture — what is parallelized, what is folded, what holds state, how data crosses each
boundary, and how the design partitions into hardware blocks. It stops at architecture: the
micro-architecture (pipeline depth, scheduling, register placement, FSM encoding) and the RTL are
decided later, by the stage that reads this document.

Write every section as prose, and keep each section on its own point. The prose carries the
architecture; everything *about* the document — its status, what has changed, and what is still open
— lives in the dedicated fields (the status line, the open-questions ledger, and the Revision log).
Two obligations apply throughout: every parameter is shown with how it was derived and where it came
from (which functional-spec section, or which requirement), and every departure from a direct
software mapping names the requirement that forced it and the number its relation produced. A
decision no reader can check does not belong here.

*Derived from: `<functional spec>` and the elicited requirements. Status: draft, pending the open
questions below.*

---

## 1. Requirements

The requirements elicited from the designer, each marked a **hard constraint** or a **target** —
throughput, clock, latency, area or target device, power, the interfaces in and out, and the
integration context. These are the inputs that shaped everything below; record the actual answers,
not the questions, and note which functional-spec `hardware-binding` entries each answer closes.

## 2. Architecture overview

The shape of the whole design in hardware: the **hardware block partition** — which need not match
the software blocks — and how the blocks connect (their interfaces and clock domains). Where the
partition departs from the functional spec's blocks (a trivial block fused, a block split to be
parallelized, a grouping by clock domain), say so and name the reason. A reader should see the whole
architecture here before reading any block in detail.

## 3. Per-block architecture

For each hardware block, its **stance** and the reasoning behind it:

- **Direct mapping**, when no requirement forces a departure — stated positively, with a sentence on
  how each relevant requirement is met as-is, plus the bindings resolved here (widths from the
  functional spec's ranges, the interface).
- **A departure**, when a relation forces one — the transformation (parallelize/unfold, fold,
  feedback storage, an interface protocol), *named beside the requirement that drove it and the
  number its relation gave*: "unfolded ×K to meet the throughput requirement [§, hard] against a
  natural cost C=…".

In either case, the block's entry **opens with one checkable line** stating the parameters its
stance rests on, each with its source:

> **Parameters:** `C = … (from <spec §>)`, `A_nat = … (from <spec §>)`, `feedback loops / T∞ = …`
> (or `none`).

This line is not optional. It is what the verify pass re-derives, and a stance argued without it —
a relation invoked but never computed — is an impression, not an architecture. Nor does the line
stand alone: the block's entry carries the **arithmetic behind it, written out** — the enumeration
that produced `C`, the operation account behind `A_nat`, each cited relation computed to its number
— because the per-block drafts this document is assembled from do not survive, and this document is
the only place a reader (or the RTL stage) can check the work. A conclusion whose derivation lives
nowhere is not shorter; it is unverifiable. After it, record the storage the block must hold (the
state carried within and across results) and the resolved bit widths.

## 4. Interfaces

The concrete protocol at each boundary — the data in, the data out, and control/status — closing the
functional spec's deferred transport bindings: the handshake or memory map, the direction, the
width, and any FIFO (with its depth) or clock-domain-crossing element the rate or domain mismatch
required.

## Open questions (ledger)

What this stage leaves open — now the micro-architectural choices deferred to the RTL stage, plus
any requirement conflict recorded as a trade rather than fudged. Give each entry a tag and a status,
the same as the functional spec's ledger, so the RTL stage inherits exactly what it must close.

## Revision log

Newest first, one line each: what changed in this document and why. The sections above always read
as the current truth; this log is where the history lives.

- `<date or version>` — <what changed, and why>.
