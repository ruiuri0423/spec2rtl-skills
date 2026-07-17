---
name: pipeline
description: >
  The coordination contract for the spec2rtl skill library. Use it to run a whole design through
  the stages (spec-recovery, hardware-spec, then block-spec): it defines how work is scheduled as a task graph,
  executed stepwise or concurrently with one invariant loop, degraded honestly on a limited harness,
  and how stages talk through the two-way ledger bus. Invoke it when orchestrating multiple blocks
  or multiple stages, when deciding what to run next, or when a question must be routed between
  stages.
---

# pipeline

The skills in this library describe *work* — recover a functional specification, give it a hardware
shape. This skill describes *how that work runs*: how it is scheduled, synchronized, parallelized,
and assigned, and how the stages talk to each other. It exists because coordination must not live
inside any one model's head. A capable model coordinates implicitly; a smaller model or a simpler
harness does not. Written down here, the coordination becomes something **any agent can follow at
whatever capability it has** — and the skills themselves stay about the work.

The division of labor is deliberate. This skill externalizes the *coordination* half of the job —
what a strong model would plan implicitly. The *reasoning* half cannot be externalized: what counts
as a block, how deep to analyze, what a value means — that stays with the model, and is kept honest
not by rules here but by each skill's obligations (show sources, compute in the open, verify,
flag-don't-fabricate). Coordination made portable; reasoning made accountable. This contract is a
floor, not a ceiling: it guarantees the minimum coordination any harness can execute, and a stronger
agent is free to plan above it, so long as the invariants below hold.

## The task graph

All work in this library runs the same way: **first draw up the schedule, then execute it in
whichever mode the harness supports.** A schedule is a set of tasks with dependencies — nothing more
exotic than that:

- A **task** is one unit of work with named inputs and outputs (specify one block; converge the
  block specs; elicit one requirement; verify one block's parameters).
- A **dependency** says a task needs another's output. Tasks with no dependency between them are
  **independent** — the schedule marks them so, because independence is what parallelism (when
  available) exploits.
- A **barrier** is a task that depends on *all* of a fan-out (aggregation-and-convergence depends
  on every block draft; the partition depends on every verified block). Barriers are how stages
  synchronize.

Schedules are **discovered, not fixed**: the first task is usually exploratory (read the entry
point, list the blocks; define the boundary), and its output determines the fan-out that
follows. Draw the schedule as far as it is known, execute, extend.

Each skill declares its own schedule in its `method.md` — spec-recovery declares
*discover → draft each block (independent) → aggregate and converge (barrier) → resolve with the
designer*; hardware-spec declares *define the boundary (per-port elicitation + reconcile) →
interface-contract barrier → per-block architecture drafts (independent) → verify each block
(independent, after its draft) → partition, aggregate, and converge (barrier)*, with a reopen path
from quantification back to the contract;
block-spec declares *decompose-and-budget each hardware block (independent) → verify each
(independent, after its draft) → aggregate and converge (barrier)*. This
skill is how any such declaration is executed.

## Drafts, deliverables, and the aggregation point

A fan-out's outputs are **drafts** — intermediate artifacts that exist to be merged — and each
stage's deliverable is **one document**: one body, one ledger, one Revision log. The rule that makes
this cheap is about *timing*: **aggregation happens at the stage's barrier, before any open question
is resolved.** Everything before the barrier is generation, which parallelizes; everything after is
resolution, which is edits — and an edit must have exactly one place to land. Aggregate too late (or
never) and every shared decision exists in N copies: every resolution costs N synchronized edits,
and a write-back task exists whose only job is keeping the copies telling one story. Aggregate at
the barrier and that cost is zero.

Concretely: a draft may travel as a returned message or a working file, whichever the harness
supports, but it does not survive the barrier as a separate document. The barrier task merges the
drafts — unifying terminology, collapsing ledger entries that are one question seen from several
blocks into a single entry naming the blocks it touches, turning cross-file references into section
references — and from then on the single document is the only artifact any task edits. The ledger
bus below routes entries between *stage documents*, one ledger per stage, never between per-block
fragments.

**Merging is reconciliation, never summarization.** What aggregation removes is *duplication* — the
N copies of a shared decision, the per-file status lines and Revision logs, the cross-file
references. What it must never remove is *substance*: everything in a draft that makes its claims
checkable — a derivation, an enumeration, the arithmetic behind a cited relation — travels into the
block's section of the document whole. The test is self-sufficiency: because drafts do not survive,
**the document must be checkable with the drafts gone**, and a deliverable that states a conclusion
while pointing at a draft as the home of its detail has destroyed the evidence, not aggregated it.
A barrier that shrinks the work into a digest has failed the same way a below-floor harness fails
by faking a full run: less is only honest when it is *reported* as less, and a summary silently
standing in for the work is not.

## Two execution modes, one loop

The same schedule runs in either mode; the outputs are identical.

**Stepwise** — the harness is a single agent that can read and write files but cannot spawn others.
Execute the tasks one at a time in dependency order, in one context: each independent task in turn,
then the barrier task once all its inputs exist. This mode requires nothing but an agent loop with
file access.

**Concurrent** — the harness can run several agents at once. Assign each independent task to its own
agent, run them simultaneously, and hold the barrier until all have returned. Assignment is
per-task: one sub-agent per block draft, one per verify pass. This mode changes wall-clock time and
nothing else.

The **loop is the same in both**: take the next task whose dependencies are satisfied → execute it →
mark it done → repeat until the schedule is drained. Synchronization points, task order at
barriers, and the final artifacts do not depend on the mode. A skill's method never assumes one mode
— it declares the schedule; the executing agent picks the mode its harness supports.

## The capability floor, and honest degradation

Running a schedule at all requires an agent that can **iterate and touch files** — read several,
write several, loop. That is the floor.

Below the floor — a single completion with no tools — the honest behavior is to execute **the first
task of the schedule only** (specify the one block in hand) and to say plainly that the rest of the
schedule was not run and what remains. What is never acceptable is producing one artifact shaped as
if the whole schedule had run. Degradation must be explicit: less work done, truthfully reported,
never silently collapsed.

## The ledger bus — how stages talk

Stages coordinate through the **open-questions ledgers** that already travel with every spec. The
ledger is not a footnote; it is the message bus of the pipeline, and it runs in **both directions**.

Two rules govern it:

1. **A stage may close only the entries tagged for it.** hardware-spec closes `hardware-binding` and
   `must-define-for-hardware`; `behavior` and `reference-defect` belong to the functional stage and
   the designer. The tags are permissions, not decoration — a stage that resolves an entry outside
   its tags has silently crossed a boundary it had no authority to cross. The tag set extends
   downstream the same way: hardware-spec's ledger defers the structural micro-architecture entries
   (`micro-arch`) to block-spec, and block-spec defers `implementation` entries to the RTL stage —
   always the same rule: a tag names the one stage with authority to close the entry.
2. **A stage that uncovers a question it cannot close routes it to its owner.** Downstream: an
   architectural spec defers micro-architecture to the RTL stage through its own ledger. Upstream: if
   hardware work exposes a *behavioral* ambiguity — an interface whose meaning the functional spec
   never pinned, a register whose intent is unclear — the hardware stage does not quietly decide it.
   It writes the entry back into the functional spec's ledger (tagged `behavior`, status `open`),
   the functional stage resolves it with the designer and updates the functional spec (recording the
   change in its Revision log), and the hardware stage then re-reads what it depends on.

The pipeline is therefore itself a loop, not a one-way street: **run a stage → route what surfaced →
re-run whichever stage now has open inputs → repeat until no ledger entry is left unrouted.** A
design is done when every entry everywhere is `resolved`, or `deferred` with its owner named.

## Driving the pipeline from outside

Nothing above requires the orchestrator to live inside any particular harness. Because schedules are
explicit tasks and stage hand-offs are ledger entries in files, an **external loop — a script, a
framework, a human — can drive the pipeline step by step**: invoke a skill for one task, inspect the
ledgers, invoke the next. A harness with concurrency runs the same pipeline faster; a harness with
none runs it one call at a time. The pipeline's correctness lives in the schedule, the barriers, and
the ledger rules — not in who happens to be turning the crank.
