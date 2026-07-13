# method — how spec-recovery reads a reference

This is the reasoning procedure the skill follows. It works in two passes over the three
things that make up any program: its **inputs and outputs**, its **data structures**, and
its **algorithm** (the control and computation flow). The first pass *understands* the
reference; the second pass *interrogates* it. Nothing is invented: anything the reference
does not settle is either resolved with the user in conversation, or written down as
`UNCONFIRMED`.

Read the two passes in order. If the understanding in Pass A is wrong, everything built on
top of it is wrong, so keep Pass A faithful and free of assumptions — save all judgement
for Pass B.

---

## Pass A — Understand (extract faithfully, do not judge yet)

Describe, in plain terms, the three essentials exactly as the code presents them.

### 1. Inputs and outputs
- The declared arguments and return values, and what each one means.
- The **hidden** inputs and outputs — the ones not in the signature but that the behavior
  depends on or produces: files it loads, global or persistent state, constant tables
  baked into the body, and values it computes but drops instead of returning. These are
  the most commonly missed and the most important to surface. The per-language patterns
  that reveal them are listed at the end of this document.
- For each input and output, describe what it *means* — what it represents and its role in
  the computation. Interpret it at the level of meaning, not representation: do not pin down
  details such as bit ordering (MSB- or LSB-first), packing, or interface form at this
  stage, even where the reference happens to imply one. Those are representational choices
  that belong to later stages, and capturing only the meaning here keeps the specification
  at the right level of abstraction.

### 2. Data structures
- Every value that holds state: scalars, arrays, tables, buffers, records.
- For each, capture what will matter later: its element type, its size or shape, how it is
  indexed, and its lifetime — does it live only within a single call (transient), or does
  it carry across calls (persistent state)? These distinctions are what eventually decide
  whether something becomes a register, a memory, or a wire, so record them now — without
  choosing any of that hardware yet. Describe each structure by what it holds and why it
  exists, not by a concrete encoding or width; the concrete form is settled later.

### 3. Algorithm flow
- The sequence of steps from inputs to outputs: the phases, the loops and what bounds
  them, the branches and the conditions that select them.
- The numeric semantics inside the steps: sign handling, saturation or wraparound,
  rounding, and fixed-point scaling. These are easy to lose and hard to recover after the
  fact, so state them precisely.

### The file's place in a larger system

A reference file is usually one part of a system: a decoder has an encoder somewhere, a
processing stage has the stage that feeds it, and the tables it loads are produced by yet
another file. Note these counterparts and dependencies as you come across them. You are
specifying only this file, but knowing what it is paired with is what keeps the next pass
honest.

Write all of this down as a faithful summary of what the reference actually does. Do not
yet ask whether it is complete or hardware-ready — that is Pass B.

---

## Pass B — Interrogate (find the gaps, then close or flag them)

Go back over the same three essentials and ask, of each, whether it is complete enough to
build from. The bar differs by essential:

- **Inputs and outputs must reach `confirmed`.** They are the boundary of the whole block
  and cannot stay vague.
- **Data structures and algorithm flow may carry open items**, but every open item must be
  made visible — never silently resolved.

Ask, in this order:

1. **Inputs and outputs — is each one's purpose and form unambiguous?** Look for unstated
   ordering, an output the caller needs but the code drops, or an input whose real form is
   only a modelling convenience (for example, an ASCII `'0'/'1'` string standing in for a
   bit stream).
2. **Data structures — is any definition incomplete in meaning?** Look for a table whose
   encoding is described only implicitly, a structure whose purpose is unclear, or state
   whose reset or initial value is undefined. Do not treat a missing representational detail
   — a width, a bit ordering — as a gap here; those are deferred to later stages by design.
3. **Algorithm flow — does any step lose information?** Look for an unbounded search loop,
   a branch with no defined alternative, an error path the code simply crashes on, or a
   boundary assumption that is never stated.

When something is uncertain or undefined, do not guess, and do not jump straight to marking
it open. First ask **where its answer would live**, and raise that question proactively.
There are three possibilities:

1. **Is there a related file?** The answer may live in another part of the system — the
   encoder that produced what this decoder consumes, the stage that feeds this one, the file
   that generates a table this one loads. If so, the point is a candidate to confirm there,
   not a settled fact here: name the file that would settle it, and read it or ask for it.
2. **Is there a related definition?** The answer may already be fixed by something known — a
   standard, a specification, or a convention the domain settles (for example, what a
   category-zero symbol means in JPEG). If so, resolve it from that definition. Where the
   definition and the code disagree, surface the conflict rather than silently trusting
   either one.
3. **Is it genuinely undefined?** If no file and no definition settles it, the point is truly
   open. Put it to the user as a single convergent question carrying a recommended answer; or,
   if no answer is available, mark it `UNCONFIRMED` in the specification, stating what is
   known and what is missing.

This ordering keeps the skill honest and makes `UNCONFIRMED` a last resort, reached only
after a related file and a known definition have both been ruled out. A correctness finding
you can only see half of is a candidate to confirm (1) or a matter of definition (2) — not a
proven defect. Reserve a firm claim for what is visibly wrong within this file alone.

### Tag every open question with where it is closed

Anything that stays open is written into the specification's open questions, and each one
carries a tag saying **which stage will close it**, so the question travels to the right place
instead of waiting to be rediscovered. There are four kinds:

- **behavior** — what the design should do; settled with the designer at spec time, and
  settling it changes the spec.
- **reference-defect** — a fault in the source model itself; routed to fixing, or knowingly
  accepting, the reference, not carried forward as if it were intended.
- **hardware-binding** — a representational or implementation choice deliberately deferred (a
  width, an interface form, a table's encoding); closed at a later hardware stage, informed by
  the ranges this spec records under Resources.
- **must-define-for-hardware** — behavior the reference left undefined that hardware cannot
  leave undefined (an illegal input, an error path, an out-of-range value); it may stay open
  here, but must be closed before the design becomes hardware.

Give each open question a status as well — `open`, `resolved`, or `deferred` with a reason — so
the list is a ledger that carries its own history rather than a note that is read once and lost.

---

## Hidden input / output patterns, by language

The signature is never the whole story. These are the places real inputs, outputs, and
state hide outside it.

| Language | Where hidden I/O and state hide |
|---|---|
| MATLAB | `readtable` / `load` / `readmatrix`; `global` / `persistent`; `evalin`, `getappdata`; values computed then never returned |
| C / C++ | `extern` variables; file I/O; global lookup tables; `#define` parameters; `static` state |
| Python | `open()` / `pd.read_*` / `np.load`; module-level constants and dicts; `os.environ`; tables pulled in by import |
| Any language | anything read from state that was not passed in; implicit default values; a table treated as a constant; state that survives between calls |

---

## Numeric semantics worth recovering precisely

These are the details most often lost between a reference model and its hardware, so state
each one as a concrete rule:

- sign reconstruction or sign extension,
- saturation versus wraparound on overflow,
- rounding mode and fixed-point scaling,
- any value that depends on the result of a previous call (cross-call prediction).

---

## Updating an existing spec

When a spec already exists and something changes — the reference is edited, a decision is settled,
a block or a file is removed — do not re-derive the whole document, and do not scatter change-notes
through the prose. Work as a focused update:

- **Find the delta.** Identify only what actually changed since the spec was written.
- **Rewrite just the sections it touches**, so each again reads as clean prose describing the
  current truth — no "(previously X)" scars, no dangling mention of what is now gone.
- **Record the change in one place**: add a newest-first line to the Revision log at the foot of
  the document, saying what changed and why.
- **Leave everything the delta did not touch exactly as it was.**

This keeps the body always-current and on its own point while the history stays in a single field.
It is distinct from the open-questions ledger: the ledger tracks the state of design *decisions*
(open → resolved / deferred); the Revision log tracks changes to the *document itself*.
