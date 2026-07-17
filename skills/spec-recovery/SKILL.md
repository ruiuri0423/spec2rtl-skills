---
name: spec-recovery
description: >
  Read a piece of reference software (MATLAB / C / C++ / Python) and reconstruct a clear,
  human-readable functional specification for it — what the block does, what goes in, what comes
  out, and the rules that connect them — before any hardware partitioning or RTL work. Use it when
  starting from an algorithm or reference model, or when taking over a design whose original intent
  was never written down.
---

# spec-recovery

A piece of reference software describes a computation, but it rarely states the *specification* a
hardware engineer needs. The author's intent is buried in the code, and is often forgotten even by
the person who wrote it. This skill reads that reference and writes the specification back out in
plain language, so that anyone who picks up the design later understands exactly what it is supposed
to do — before a single line of RTL is written.

Two convictions run through everything the skill does. The first is **human-readability**: the
specification, and this skill itself, must be understandable by a person — prose over fragmented
tables, and output kept within a range someone can actually hold in their head. The second is that
the skill **drives a model's reasoning rather than dictating the shape of its output**: there are no
rules here about length or wording. What is required instead is that every statement be *true* —
traceable to the reference, never invented — and *reasoned* rather than guessed. Those are the
invariants; the structure exists only so the result reads clearly and hands forward consistently.

## What this skill does

It recovers the specification hidden inside a behavioral reference and states it plainly: the
block's purpose in one honest sentence, each input and output and what it means, the rules that turn
inputs into outputs, and — just as important — the places where the original intent is unclear or
missing, raised for a decision rather than quietly filled in. The result should read like an
explanation a colleague would give, not a machine dump.

## What this skill reads

It reads the one reference file the user points to. Crucially, it looks past the function signature
to the inputs the code depends on but never declares — files it loads, global tables, constants
baked into the body, values it computes but drops. In hardware these are real inputs, and whoever
inherits the block cannot use it without knowing they exist; surfacing them is one of the most
valuable things the skill does. Because it reads only that one file, it is honest about what it
cannot see in full: when something is uncertain, it asks *where the answer would live* — in a
related file, in a known definition, or nowhere at all — rather than asserting a fact it has only
half seen.

## What this skill delivers

One specification document, `<source>.spec.md`, written as prose a person can read start to finish:
what the block does, its inputs and outputs and what they mean, how it behaves, and an
**open-questions ledger** of what a human still has to decide. When the reference is a whole design
of several files, the deliverable is still **one document** — top-level sections that tie the design
together, a per-block section for each block, and a single merged ledger and Revision log — so every
decision has exactly one home. The per-block work happens as parallel drafts, but drafts are
intermediates: they are aggregated at the converge barrier, before anything is resolved with the
designer. When the design is headed to hardware, the document closes with a **hardware-ization
scope** section — which parts of the reference go to hardware, and how that hardware is expected to
sit in the surrounding system — settled with the designer as the stage's closing conversation, so
the hardware stage inherits its scope instead of re-deriving it. What the document should contain
is set out in `references/spec-template.md`.

## How it works

The full procedure is in `references/method.md`; in outline:

- **Understand, then interrogate.** A first pass states faithfully what the reference does — its
  inputs and outputs, its data structures, its algorithm — capturing *meaning*, not representation
  (bit widths, ordering, and interface form are named as deferred, never pinned). A second pass asks
  where each part is still incomplete.
- **Route every uncertainty by where its answer lives** — a related file, a known definition, or
  genuinely undefined — so guessing is never the move and `UNCONFIRMED` is a last resort.
- **Carry what stays open as a tagged ledger.** Each open question names the stage that will close
  it, and is resolved with the designer in a convergent conversation, one question at a time, each
  with a recommended answer.
- **End by scoping the hardware.** Once the behavior has converged, two closing questions to the
  designer: which parts of the reference are to be **hardware-ized** (and which stay software — the
  test harness, file I/O, golden dumping never are), and how the hardware-ized part **connects to
  the surrounding architecture** (who feeds it, who consumes it, who configures it). Asked here
  because this is the moment both parties understand the behavior best; recorded in the scope
  section and inherited by the hardware stage.
- **Specify a whole design as a schedule** — discover the blocks, draft each in parallel
  (independent tasks), then aggregate and converge at a barrier into the one deliverable document
  *before* resolving anything with the designer. How the schedule runs — stepwise in one context or
  concurrently across agents — is the harness's choice, governed by the library's coordination
  contract — the `pipeline` skill; the outputs are identical either way.
- **Record updates in one place** — a Revision log — rather than scattering change-notes through the
  prose.

## What this skill must not do

It does not divide the design into hardware blocks, choose a micro-architecture, interface, or bit
width, or generate RTL — those belong to later stages that read this specification. It does not
invent behavior the reference does not show: where something is genuinely undefined it raises the
question. And it never fabricates to fill a section — on a point it cannot settle, it says so, so
the specification stays true even where it is incomplete.
