# Block Implementation Specification — <design or block name>

This document is the output of the block-spec stage: for each hardware block, the sub-modules it is
built from, the contract each must satisfy, and the budget each may spend — allocated top-down from
the requirements so that implementation can proceed bottom-up, each sub-module provable against its
own contract before anything is integrated. It stops at contracts and budgets: the RTL, and the
coding-level choices inside a sub-module (FSM encoding, register placement within a stage, reset
detail), are decided by the stage that reads this document.

Write every section as prose, and keep each section on its own point. Two obligations apply
throughout: every budget is shown with the arithmetic that allocated it and the source it was
inherited from, and every allocation sums — to the block's envelope, to the path's latency — with
slack stated. A contract a reader cannot check does not belong here.

*Derived from: `<hardware spec>` (and the functional spec's rules, restated where contracts need
them). Status: draft, pending the open questions below.*

---

## 1. Inherited requirements and block envelopes

The design-level requirements and each hardware block's envelope — clock, initiation interval, area
envelope, latency, state — as the hardware spec derived them, each with its source. Restated here
so the document stands alone; nothing new is decided in this section.

## 2. Budget allocation overview

The allocation tree, shown once for the whole design: hardware block → sub-module, with the
arithmetic that splits each envelope and the sums that close it. A reader should be able to audit
every number from this section and §1 alone. Slack, and any allocation that does not close (a
routed trade), is stated here, not hidden inside a chapter.

## Per hardware block: <block name>

One chapter per hardware block. Open with the block's sub-module map — the sub-modules, the
interfaces between them, and which carries which share of the block's budgets — then one contract
per sub-module:

### Sub-module: <name>

> **Budget:** `II = … (from …)`, `T_comb ≤ … per stage over S = … stages (…)`, `A ≤ … (…)`,
> `state = … (owned here)`.

- **Function** — what it computes, with the exact numeric rules (rounding, clamping, scaling, sign
  handling) carried whole from the functional spec, and the section they come from. Restate; do not
  paraphrase.
- **Interface** — ports, widths from the hardware spec's bindings, and the advance rule between
  sub-modules — inherited from the requirements' streaming discipline — that preserves the block's
  II under composition.
- **State and reset** — what it owns, how long it lives, what reset leaves it as.
- **Verification obligation** — the golden evidence that will prove the implementation: exhaustive
  (with the bound stated), reference-dump (naming the files and the boundary they prove), and any
  independent-equivalence evidence — with the pass criterion.

## Interfaces between blocks

The block-boundary interfaces the hardware spec pinned, refined here only where the sub-module
decomposition sharpened them — the timing of data advance at the boundary, under the streaming
discipline the requirements set. Nothing here may contradict the hardware spec; a needed change
routes back through the ledger.

## Open questions (ledger)

What this stage leaves open — the coding-level entries deferred to the RTL stage, any allocation
recorded as a trade rather than fudged, and entries routed to other stages with their owners named.
Tag and status on every entry, as everywhere in the library.

## Revision log

Newest first, one line each: what changed in this document and why. The sections above always read
as the current truth; this log is where the history lives.

- `<date or version>` — <what changed, and why>.
