---
name: spec-recovery
description: >
  Read a piece of reference software (MATLAB, C, C++, or Python) and reconstruct a
  clear, human-readable functional specification for it — what the block does, what
  goes in, what comes out, and the rules that connect them — before any hardware
  partitioning or RTL work. Use it when starting from an algorithm or reference model,
  or when taking over a design whose original intent was never written down.
---

# spec-recovery

A piece of reference software describes a computation, but it rarely states the
*specification* a hardware engineer needs. The author's intent is buried in the code,
and is often forgotten even by the person who wrote it. This skill reads that reference
and writes the specification back out in plain language, so that anyone who picks up the
design later understands exactly what it is supposed to do — before a single line of RTL
is written.

The guiding idea is that both the specification and this skill itself must be
understandable by a human. Prefer clear, connected prose over fragmented lists, and keep
the output within a range a person can actually read and act on. A specification nobody
can hold in their head is not doing its job.

## What this skill does

It recovers the specification hidden inside a behavioral reference and states it plainly.
It names the block's purpose in one honest sentence; it describes each input and output
and what it means; it writes down the rules that turn inputs into outputs; and — just as
important — it points out the places where the original intent is unclear or missing, so
the user can decide, rather than the skill quietly guessing. The result should read like
an explanation a colleague would give, not a machine dump.

## What this skill reads

It reads the one reference file the user points to. Crucially, it looks past the function
signature to the inputs the code depends on but never declares — files it loads, global
tables, constants baked into the body. In hardware these are real inputs, and a person
who inherits the block cannot use or change it without knowing they exist. Surfacing
these hidden dependencies is one of the most valuable things this skill does.
`references/method.md` lists, per language, the patterns that reveal them.

Because it reads only that one file, it is honest about what it cannot see in full. Whenever
something is uncertain, it asks where the answer would live — in a related file (the encoder
behind a decoder, the source of a table), in a known definition or standard, or nowhere at
all — and only what none of these can settle becomes a genuinely open question. A point it
cannot resolve from the file alone is something to confirm or to leave open, never a fact it
has proven. `references/method.md` sets out this triage in full.

## What this skill delivers

One specification document, named `<source>.spec.md`, written so a person can read it
from start to finish and come away understanding the design. What that document should
contain is described in full in `references/spec-template.md` —
follow it. The specification states what the block does, what its inputs and outputs are
and what they mean, how it behaves, and where a human decision is still needed. Where the
code forces a choice that the hardware will eventually need but that is not part of the
block's essential behavior, record it as an open decision instead of baking it in. When the
reference is a whole design of several blocks rather than a single one, the skill delivers one
such specification per block together with a top-level spec that ties them together, as
described under "Working across a whole design" below.

## How the specification is completed

The work produces a *first version* of the specification: complete in structure, but still
carrying the open questions the reference could not settle. The skill does not stop there,
and it does not leave those questions as a silent list at the bottom of the page. It works
through them with the user in a convergent conversation — one question at a time, each
phrased plainly and each carrying a recommended answer drawn from the code or the domain,
so the user is deciding between clear options rather than starting from a blank page. As
each question is answered, the specification is updated in place: that point moves out of
the open questions and into the body. When the questions are resolved — or a deliberate few
are left open on purpose — the result is the *complete version* of the specification. The
intent is that the user arrives at a trustworthy spec through a short series of small, clear
decisions, never by facing a wall of choices at once.

Not every question is settled here, and that is expected. Each open question is tagged with the
stage that will close it — a matter of design intent settled now with the designer, a defect in
the reference to be fixed, a hardware choice deliberately deferred, or behavior that hardware
cannot leave undefined — and is carried forward as a small ledger with a status, not a footnote.
A later stage inherits the questions tagged for it and must close them, so nothing is left to be
rediscovered by re-reading the whole spec. (`references/method.md` defines the four tags.)

## Working across a whole design

A reference is often not a single file but a whole design — a project of many files, each a
block with its own job, that together form a pipeline. JPEG is like this: a driver calls a
forward transform, a quantizer, an entropy coder, and their inverses, each living in its own
file. To carry such a design toward hardware, every block deserves the same specification this
skill gives a single one — its purpose, its inputs and outputs, its data structures, its
behavior — so that the hardware can be built up from the blocks, bottom-up, against specs that
are already clear and connected.

The skill handles this by fanning out: it specifies the blocks in parallel, draws the results
back together, and then pushes the settled results back down to the blocks. It works in five
movements.

1. **Sketch a first-version top-level spec.** Read the entry point — the driver or top
   function — to understand what the whole design does, and to identify the blocks it is made
   of and how they connect. These are the software's *own* blocks — its files and functions —
   not a hardware partition invented here; that decision belongs to a later stage. The result
   is a first, still-provisional specification of the whole, whose behavior is the pipeline of
   blocks.
2. **Specify every block in parallel.** Give each block to its own sub-agent, which runs this
   same method on that one file — Pass A, then Pass B, with the three-route triage for
   anything uncertain — and returns that block's specification. Running the sub-agents in
   parallel is what makes specifying a large design practical, and because every one follows
   the same method, the block specs come back consistent with each other and with a
   single-block spec.
3. **Converge the block specs.** Bring the parallel results together and reconcile them where
   the blocks meet: does one block's output actually match, in meaning, the input the next
   block expects? This is where the "related file" route pays off — a question left open in
   one block is often answered by a sibling's spec, and a disagreement between two of them is
   a conflict to surface rather than smooth over. What is still open after this is a genuine
   system-level question for the user.
4. **Converge the top-level spec.** Fold the now-confirmed understanding of the blocks back
   into the top-level spec, so that it is consistent with its parts and the questions only the
   whole design could answer are settled.
5. **Propagate the resolutions back to the blocks.** Convergence settles things — a block's
   open question answered by a sibling, a ledger entry moved from open to resolved or deferred,
   a cross-reference between two blocks. Those results must flow *back down* into the individual
   block specs, or a block is left saying "draft, pending" while the system around it has moved
   on. This write-back parallelizes the same way the specifying did: give each block to its own
   sub-agent to update its status line, close the local ledger entries convergence settled, and
   add the cross-references — so the whole set tells one consistent story, top-level and blocks
   agreeing rather than drifting apart.

What this produces is a small library — one specification per block, plus a top-level spec that
ties them together, all left mutually consistent. That is exactly the form a bottom-up hardware
effort needs: each block can be built against a clear spec of its own, while the top-level spec
holds the integration together.

## What this skill must not do

It does not divide the design into hardware blocks, does not choose a micro-architecture,
interface, or bit width, and does not generate RTL. Those belong to later stages that
read this specification. It also does not invent behavior the reference does not show:
where something is genuinely undefined, it raises the question rather than filling the gap
on its own.
